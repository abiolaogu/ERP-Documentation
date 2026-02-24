# ERP-Autonomous-Coding -- Git Bridge Service Specification

## Document Information

| Field | Value |
|-------|-------|
| Service | git-bridge |
| Language | Go 1.22 |
| Port | Internal only |
| Source | `/services/git-bridge/` |

---

## 1. Service Overview

Git Bridge provides a unified abstraction layer over four Git hosting providers: GitHub, GitLab, Bitbucket, and Azure DevOps. It handles the complete PR lifecycle, webhook event processing, and the AIDD human approval gate.

```mermaid
flowchart TB
    subgraph "Git Bridge"
        GRPC2["gRPC Server"]
        WEBHOOK3["Webhook HTTP Server"]
        ABSTRACT2["Unified Git Interface"]
        AIDD2["AIDD Approval Gate"]
        CRED["Credential Manager"]
        QUEUE3["Event Queue"]
    end

    subgraph "Provider Adapters"
        GH2["GitHub Adapter<br/>(App + REST v3 + GraphQL v4)"]
        GL2["GitLab Adapter<br/>(OAuth + REST v4 + GraphQL)"]
        BB2["Bitbucket Adapter<br/>(App + REST v2)"]
        AZ2["Azure DevOps Adapter<br/>(OAuth + REST v7.1)"]
    end

    GRPC2 --> ABSTRACT2
    WEBHOOK3 --> QUEUE3
    ABSTRACT2 --> GH2
    ABSTRACT2 --> GL2
    ABSTRACT2 --> BB2
    ABSTRACT2 --> AZ2
    ABSTRACT2 --> AIDD2
    ABSTRACT2 --> CRED
```

---

## 2. Unified Git Interface

### 2.1 Interface Definition

```go
type GitProvider interface {
    // Repository operations
    CloneRepository(ctx context.Context, req CloneRequest) (*CloneResult, error)
    GetRepository(ctx context.Context, owner, name string) (*Repository, error)

    // Branch operations
    CreateBranch(ctx context.Context, req CreateBranchRequest) (*Branch, error)
    DeleteBranch(ctx context.Context, req DeleteBranchRequest) error

    // Commit operations
    Commit(ctx context.Context, req CommitRequest) (*Commit, error)
    Push(ctx context.Context, req PushRequest) error

    // Pull Request operations
    CreatePullRequest(ctx context.Context, req CreatePRRequest) (*PullRequest, error)
    UpdatePullRequest(ctx context.Context, req UpdatePRRequest) (*PullRequest, error)
    GetPullRequest(ctx context.Context, id string) (*PullRequest, error)
    ListPullRequests(ctx context.Context, req ListPRRequest) ([]*PullRequest, error)
    MergePullRequest(ctx context.Context, req MergePRRequest) error
    PostReviewComment(ctx context.Context, req ReviewCommentRequest) error
    RespondToReview(ctx context.Context, req RespondRequest) error

    // Webhook operations
    RegisterWebhook(ctx context.Context, req WebhookRequest) (*Webhook, error)
    VerifyWebhook(ctx context.Context, req VerifyRequest) (bool, error)

    // CI/CD operations
    GetPipelineStatus(ctx context.Context, req PipelineRequest) (*PipelineStatus, error)
    TriggerPipeline(ctx context.Context, req TriggerRequest) (*Pipeline, error)
}
```

### 2.2 Provider Adapter Pattern

```mermaid
flowchart TB
    CALLER["Agent Core / Review Engine"] --> INTERFACE["GitProvider Interface"]

    INTERFACE --> FACTORY["Provider Factory"]
    FACTORY -->|provider=github| GH3["GitHubAdapter"]
    FACTORY -->|provider=gitlab| GL3["GitLabAdapter"]
    FACTORY -->|provider=bitbucket| BB3["BitbucketAdapter"]
    FACTORY -->|provider=azure-devops| AZ3["AzureDevOpsAdapter"]

    GH3 --> GH_REST["GitHub REST v3"]
    GH3 --> GH_GQL["GitHub GraphQL v4"]
    GH3 --> GH_APP2["GitHub App Auth"]

    GL3 --> GL_REST["GitLab REST v4"]
    GL3 --> GL_GQL["GitLab GraphQL"]
    GL3 --> GL_OAUTH["GitLab OAuth2"]

    BB3 --> BB_REST["Bitbucket REST v2"]
    BB3 --> BB_APP2["Bitbucket App Auth"]

    AZ3 --> AZ_REST["Azure DevOps REST v7.1"]
    AZ3 --> AZ_OAUTH["Azure AD OAuth2"]
```

---

## 3. Provider Feature Matrix

| Feature | GitHub | GitLab | Bitbucket | Azure DevOps |
|---------|--------|--------|-----------|-------------|
| Clone via HTTPS | Yes | Yes | Yes | Yes |
| Clone via SSH | Yes | Yes | Yes | Yes |
| REST API | v3 | v4 | v2 | v7.1 |
| GraphQL API | v4 | Yes | No | No |
| App-based Auth | GitHub App | OAuth2 | App Password | OAuth2 |
| Webhooks | Yes | Yes | Yes | Service Hooks |
| PR/MR Management | Yes (PRs) | Yes (MRs) | Yes (PRs) | Yes (PRs) |
| CI/CD Integration | Actions | GitLab CI | Pipelines | Azure Pipelines |
| Issue Tracking | Issues | Issues | Jira integration | Work Items |
| Review Comments | Yes | Yes | Yes | Yes (Threads) |
| Status Checks | Check Runs | Pipeline status | Build status | Policy checks |
| Copilot Bridge | Yes | N/A | N/A | N/A |

---

## 4. AIDD Approval Gate

```mermaid
sequenceDiagram
    participant Agent as Agent Core
    participant GB as Git Bridge
    participant Provider as Git Provider
    participant Human as Human Reviewer
    participant KF as Kafka

    Agent->>GB: CreatePullRequest(...)
    GB->>Provider: Create PR/MR
    Provider-->>GB: PR URL
    GB->>KF: Emit pr.created
    GB->>KF: Emit pr.approval_required

    Note over Human: Notification sent

    Human->>Provider: Review PR, add comments
    Provider->>GB: Webhook: review submitted
    GB->>KF: Emit webhook.received

    alt Approved
        Human->>Provider: Approve PR
        Provider->>GB: Webhook: review approved
        GB->>GB: Check AIDD policy (min approvals met?)
        GB->>Provider: Merge PR
        GB->>KF: Emit pr.approved
        GB->>KF: Emit pr.merged
    else Changes Requested
        Human->>Provider: Request changes
        Provider->>GB: Webhook: changes requested
        GB->>Agent: Changes requested notification
        Agent->>Agent: Address feedback
        Agent->>GB: Update PR (push new commits)
    else Rejected
        Human->>Provider: Close PR
        Provider->>GB: Webhook: PR closed
        GB->>KF: Emit pr.rejected
    end
```

---

## 5. Webhook Processing

### 5.1 Webhook Verification

| Provider | Verification Method | Header |
|----------|-------------------|--------|
| GitHub | HMAC-SHA256 of payload | `X-Hub-Signature-256` |
| GitLab | Secret token comparison | `X-Gitlab-Token` |
| Bitbucket | IP allowlist + secret | N/A (IP-based) |
| Azure DevOps | Basic auth or OAuth | `Authorization` |

### 5.2 Webhook Processing Pipeline

```mermaid
flowchart LR
    RECEIVE["Receive webhook"] --> VERIFY["Verify signature"]
    VERIFY --> PARSE["Parse event type"]
    PARSE --> NORMALIZE["Normalize to internal event"]
    NORMALIZE --> PUBLISH["Publish to Kafka"]
    PUBLISH --> ACK["Acknowledge (200 OK)"]

    VERIFY -->|Invalid| REJECT["Reject (401)"]
```

---

## 6. Credential Management

```mermaid
flowchart TB
    subgraph "Credential Flow"
        REQUEST["Git operation request"] --> RESOLVE["Resolve credentials"]
        RESOLVE --> CACHE_CHECK{In Redis cache?}
        CACHE_CHECK -->|Yes, not expired| USE["Use cached credential"]
        CACHE_CHECK -->|No or expired| VAULT["Fetch from Vault"]
        VAULT --> REFRESH{Token expired?}
        REFRESH -->|Yes| RENEW["Refresh OAuth token"]
        RENEW --> STORE["Update Vault + cache"]
        REFRESH -->|No| STORE
        STORE --> USE
        USE --> OPERATION["Execute Git operation"]
    end
```

Credentials are never stored in the database. They are managed exclusively through HashiCorp Vault with short-lived tokens cached in Redis.

---

## 7. GitHub Copilot Bridge

For organizations using GitHub Copilot alongside ERP-Autonomous-Coding, the Copilot Bridge provides:

- Shared context between Copilot suggestions and agent sessions
- Agent can reference and extend Copilot-generated code
- Unified telemetry for both Copilot and autonomous agent usage
- Policy coordination (e.g., content filters apply to both)

---

## 8. Error Handling

| Error | Retry | Fallback |
|-------|-------|----------|
| API rate limit (429) | Backoff per Retry-After header | Queue operations |
| Auth token expired | Auto-refresh, retry once | Re-authenticate |
| Webhook delivery failure | GitHub auto-retries (3x) | Dead letter queue |
| Clone timeout | Retry with shallow clone | Fail session |
| Merge conflict | Notify agent for rebase | Manual resolution |
| Provider API outage | Retry with exponential backoff | Queue and retry later |
