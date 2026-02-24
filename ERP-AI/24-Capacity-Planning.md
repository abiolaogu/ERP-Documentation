# ERP-AI Capacity Planning Guide

| Field | Value |
|---|---|
| Module | ERP-AI |
| Version | 1.0.0 |
| Last Updated | 2026-02-23 |

---

## 1. Resource Sizing

### 1.1 Small (< 500 users)

| Component | Instances | CPU | Memory |
|---|---|---|---|
| Copilot Service | 2 | 1 vCPU | 2 GB |
| Agent Orchestrator | 2 | 500m | 1 GB |
| Other Services | 1 each | 250m | 512 MB |
| Qdrant | 1 | 4 vCPU | 16 GB |
| PostgreSQL | 1 | 2 vCPU | 8 GB |
| Redis | 1 | 1 vCPU | 4 GB |

### 1.2 Medium (500-5,000 users)

| Component | Instances | CPU | Memory |
|---|---|---|---|
| Copilot Service | 4 | 2 vCPU | 4 GB |
| Agent Orchestrator | 3 | 1 vCPU | 2 GB |
| NLP Service | 3 | 1 vCPU | 2 GB |
| Embedding Service | 3 | 1 vCPU | 4 GB |
| Other Services | 2 each | 500m | 1 GB |
| Qdrant | 3 (cluster) | 8 vCPU | 32 GB |
| PostgreSQL | 2 | 4 vCPU | 16 GB |
| Redis | 3 (cluster) | 2 vCPU | 8 GB |

### 1.3 Large (5,000-50,000 users)

| Component | Instances | CPU | Memory |
|---|---|---|---|
| Copilot Service | 8 | 4 vCPU | 8 GB |
| Agent Orchestrator | 4 | 2 vCPU | 4 GB |
| NLP Service | 6 | 2 vCPU | 4 GB |
| Embedding Service | 6 | 2 vCPU | 8 GB |
| Other Services | 3 each | 1 vCPU | 2 GB |
| Qdrant | 6 (sharded) | 16 vCPU | 64 GB |
| PostgreSQL | 3 | 8 vCPU | 32 GB |
| Redis | 6 (cluster) | 4 vCPU | 16 GB |

---

## 2. Scaling Triggers

| Metric | Threshold | Action |
|---|---|---|
| Copilot latency p95 > 500ms | 5 min sustained | Scale copilot +2 |
| Agent pod queue > 50 | 2 min sustained | Scale orchestrator +1 |
| Qdrant memory > 80% | Alert | Add Qdrant node |
| Claude API rate limit | Approaching | Request quota increase |
| NLP queue depth > 100 | 5 min sustained | Scale NLP +2 |

---

## 3. Cost Projections

### 3.1 Infrastructure Cost

| Tier | Monthly Infra | Claude API | Total |
|---|---|---|---|
| Small | $1,200 | $500 | $1,700 |
| Medium | $4,000 | $3,000 | $7,000 |
| Large | $15,000 | $15,000 | $30,000 |

### 3.2 Claude API Cost Model

| Usage | Tokens/month | Cost |
|---|---|---|
| Light (copilot only) | 1M | $15 |
| Medium (copilot + agents) | 10M | $150 |
| Heavy (full AI features) | 100M | $1,500 |
| Enterprise | 1B | $12,000 |

---

## 4. Vector Storage Growth

| Vectors | Qdrant Memory | Disk |
|---|---|---|
| 1M | 8 GB | 12 GB |
| 10M | 80 GB | 120 GB |
| 100M | 800 GB | 1.2 TB |
