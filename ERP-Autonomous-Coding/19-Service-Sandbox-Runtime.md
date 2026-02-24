# ERP-Autonomous-Coding -- Sandbox Runtime Service Specification

## Document Information

| Field | Value |
|-------|-------|
| Service | sandbox-runtime |
| Language | Go 1.22 |
| Port | Internal only |
| Source | `/services/sandbox-runtime/` |

---

## 1. Service Overview

Sandbox Runtime manages the lifecycle of ephemeral Docker containers that provide isolated execution environments for agent-generated code. It handles container creation, resource limit enforcement, code execution, filesystem snapshotting, and container cleanup.

```mermaid
flowchart TB
    subgraph "Sandbox Runtime"
        GRPC["gRPC Server"]
        MGR["Container Manager"]
        POOL["Warm Pool Controller"]
        EXEC["Execution Engine"]
        SNAP["Filesystem Snapshotter"]
        LIMITS["Resource Enforcer"]
        NET2["Network Policy Manager"]
    end

    GRPC --> MGR
    MGR --> POOL
    MGR --> EXEC
    MGR --> SNAP
    MGR --> LIMITS
    MGR --> NET2

    POOL --> DOCKER["Docker Engine"]
    EXEC --> DOCKER
    LIMITS --> CGROUPS["cgroups v2"]
    NET2 --> IPTABLES["iptables / nftables"]
```

---

## 2. Container Lifecycle

```mermaid
stateDiagram-v2
    [*] --> Warm: Pre-created (pool)
    [*] --> Creating: On-demand
    Creating --> Ready: Image pulled, container started
    Warm --> Assigned: Agent claims container
    Ready --> Assigned: Agent claims container
    Assigned --> Executing: Command submitted
    Executing --> Idle: Command completed
    Idle --> Executing: Next command
    Idle --> Snapshotting: Snapshot requested
    Snapshotting --> Idle: Snapshot complete
    Assigned --> Destroying: Session complete
    Destroying --> [*]: Container removed

    Executing --> Killed: Resource limit breached
    Executing --> Killed: Timeout exceeded
    Killed --> Destroying
```

---

## 3. Warm Pool Architecture

```mermaid
flowchart TB
    subgraph "Warm Pool"
        CTRL["Pool Controller"]
        Q_GO["Go Queue (3 warm)"]
        Q_PY["Python Queue (5 warm)"]
        Q_NODE["Node.js Queue (3 warm)"]
        Q_RUST["Rust Queue (2 warm)"]
        Q_JAVA["Java Queue (2 warm)"]
        Q_NET[".NET Queue (2 warm)"]
    end

    CTRL --> Q_GO
    CTRL --> Q_PY
    CTRL --> Q_NODE
    CTRL --> Q_RUST
    CTRL --> Q_JAVA
    CTRL --> Q_NET

    subgraph "Pool Policy"
        MIN["Min warm: configurable per image"]
        MAX["Max total: configurable (default 100)"]
        TTL["Warm TTL: 10 minutes"]
        SCALE["Auto-scale: based on demand"]
    end

    CTRL --> MIN
    CTRL --> MAX
    CTRL --> TTL
    CTRL --> SCALE
```

The warm pool maintains pre-started containers for each language image. When an agent session requests a sandbox, a warm container is assigned instantly (< 1s) rather than starting cold (< 5s). The pool controller monitors demand patterns and pre-scales during peak usage hours.

---

## 4. gRPC API

```protobuf
service SandboxService {
  // Create a new sandbox container
  rpc CreateSandbox(CreateSandboxRequest) returns (CreateSandboxResponse);

  // Execute a command in a sandbox
  rpc Execute(ExecuteRequest) returns (ExecuteResponse);

  // Stream execution output in real-time
  rpc StreamExecute(ExecuteRequest) returns (stream ExecuteOutput);

  // Take a filesystem snapshot
  rpc Snapshot(SnapshotRequest) returns (SnapshotResponse);

  // Get sandbox status
  rpc GetStatus(GetStatusRequest) returns (SandboxStatus);

  // Destroy a sandbox
  rpc Destroy(DestroyRequest) returns (DestroyResponse);

  // Get pool statistics
  rpc GetPoolStats(Empty) returns (PoolStats);
}

message CreateSandboxRequest {
  string session_id = 1;
  string image = 2;          // e.g., "erp/sandbox-python:3.12"
  ResourceLimits limits = 3;
  string network_policy = 4;  // isolated, registry_only, allowlist
  repeated string allowlist_domains = 5;
}

message ResourceLimits {
  int32 cpu_cores = 1;        // max CPU cores
  int64 memory_mb = 2;        // max memory in MB
  int64 disk_mb = 3;          // max disk in MB
  int32 max_pids = 4;         // max processes
  int32 timeout_seconds = 5;  // execution timeout
}

message ExecuteRequest {
  string sandbox_id = 1;
  string command = 2;
  string working_dir = 3;
  map<string, string> env = 4;
  int32 timeout_seconds = 5;
}

message ExecuteResponse {
  int32 exit_code = 1;
  string stdout = 2;
  string stderr = 3;
  int64 duration_ms = 4;
  ResourceUsage resource_usage = 5;
}
```

---

## 5. Resource Enforcement

| Resource | Mechanism | Default Limit | Hard Maximum |
|----------|-----------|---------------|-------------|
| CPU | cgroups v2 cpu.max | 2 cores | 4 cores |
| Memory | cgroups v2 memory.max | 4 GB | 8 GB |
| Disk | overlay quota / tmpfs size | 10 GB | 20 GB |
| PIDs | cgroups v2 pids.max | 256 | 512 |
| Network | iptables rules + nftables | Deny all | Allowlist |
| Time | Process timeout + SIGKILL | 300s | 600s |

```mermaid
flowchart TB
    CMD["Command Execution"] --> MONITOR["Resource Monitor<br/>(every 500ms)"]
    MONITOR --> CPU_CHECK{CPU > limit?}
    CPU_CHECK -->|Yes| THROTTLE["Throttle via cgroups"]
    CPU_CHECK -->|No| MEM_CHECK{Memory > limit?}
    MEM_CHECK -->|Yes| OOM_KILL["OOM Kill Container"]
    MEM_CHECK -->|No| DISK_CHECK{Disk > limit?}
    DISK_CHECK -->|Yes| READONLY["Make fs read-only"]
    DISK_CHECK -->|No| TIME_CHECK{Time > timeout?}
    TIME_CHECK -->|Yes| SIGKILL["SIGKILL + cleanup"]
    TIME_CHECK -->|No| MONITOR
```

---

## 6. Filesystem Snapshotter

The snapshotter captures before/after filesystem state for audit purposes:

```mermaid
flowchart LR
    BEFORE["Before Snapshot<br/>(on sandbox create)"] --> DIFF["Diff Calculator"]
    AFTER["After Snapshot<br/>(on session complete)"] --> DIFF
    DIFF --> RESULT["Filesystem Diff"]
    RESULT --> FILES_ADDED["Files Added"]
    RESULT --> FILES_MODIFIED["Files Modified"]
    RESULT --> FILES_DELETED["Files Deleted"]
    RESULT --> STORE["Store to S3<br/>(with TTL)"]
```

---

## 7. Security Configuration

```yaml
# Sandbox container security settings
security:
  runtime: "runsc"  # gVisor
  seccomp_profile: "restricted"
  apparmor_profile: "sandbox-restricted"
  readonly_rootfs: true
  no_new_privileges: true
  user: "1000:1000"  # Non-root
  capabilities:
    drop: ["ALL"]
    add: ["NET_BIND_SERVICE"]
  tmpfs:
    - path: "/tmp"
      size: "1g"
    - path: "/workspace"
      size: "10g"
  volumes:
    # No host mounts allowed
    host_mounts: false
```

---

## 8. Metrics

| Metric | Type | Description |
|--------|------|-------------|
| `sandbox_pool_warm_total` | Gauge | Warm containers per image |
| `sandbox_pool_active_total` | Gauge | Active containers |
| `sandbox_creation_duration_seconds` | Histogram | Container creation time |
| `sandbox_execution_duration_seconds` | Histogram | Command execution time |
| `sandbox_resource_cpu_usage` | Gauge | CPU usage per container |
| `sandbox_resource_memory_bytes` | Gauge | Memory usage per container |
| `sandbox_resource_limit_breaches_total` | Counter | Resource limit violations |
| `sandbox_oom_kills_total` | Counter | OOM kills |
| `sandbox_timeouts_total` | Counter | Execution timeouts |
