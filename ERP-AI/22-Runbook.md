# ERP-AI Operational Runbook

| Field | Value |
|---|---|
| Module | ERP-AI |
| Audience | SRE / DevOps |
| Version | 1.0.0 |
| Last Updated | 2026-02-23 |

---

## 1. Service Health Checks

```bash
for svc in agent-orchestrator agent-catalog nlp-service ml-pipeline-service embedding-service guardrail-service copilot-service; do
  echo "Checking $svc..."
  curl -s http://$svc:8080/healthz | jq .
done
```

---

## 2. Common Incidents

### 2.1 Claude API Outage

**Symptoms**: Copilot suggestions failing, NLP errors, agent task failures

**Diagnosis**:
```bash
# Check Claude API status
curl -s https://status.anthropic.com/api/v2/status.json | jq .

# Check error rate
kubectl logs -l app=copilot-service -n ai-system --tail=50 | grep "claude"
```

**Resolution**:
1. Confirm external outage
2. Enable fallback mode (cached responses, simpler models)
3. Notify users via status page
4. Monitor for recovery

### 2.2 Qdrant Memory Pressure

**Symptoms**: Slow vector search, embedding indexing failures

**Diagnosis**:
```bash
# Check Qdrant collections
curl http://qdrant:6333/collections | jq '.result.collections[] | {name, vectors_count}'

# Check memory usage
kubectl top pod -n ai-data -l app=qdrant
```

**Resolution**:
1. Check for orphaned vectors (deleted tenants)
2. Compact collections: `POST /collections/{name}/points/scroll` with cleanup
3. Scale Qdrant cluster if needed
4. Increase memory limits

### 2.3 Agent Pod Failures

**Symptoms**: Agent execution errors, timeout errors

**Diagnosis**:
```bash
# Check agent pod status
kubectl get pods -n ai-agents --sort-by=.status.startTime

# Check failed pods
kubectl get pods -n ai-agents --field-selector=status.phase=Failed

# Check logs of failed pod
kubectl logs <pod-name> -n ai-agents
```

**Resolution**:
1. Check for OOMKilled events
2. Increase resource limits for failing agent type
3. Check for circular agent chains
4. Reset agent memory if corrupted

### 2.4 Model Drift

**Symptoms**: Declining prediction accuracy, user complaints

**Diagnosis**:
```bash
# Check model metrics
kubectl exec -it ml-pipeline-0 -n ai-system -- curl localhost:8083/v1/ml-pipeline/metrics
```

**Resolution**:
1. Verify drift score and affected model
2. Trigger retraining with fresh data
3. Evaluate new model against baseline
4. Deploy via canary if improved
5. Rollback if regression

---

## 3. Scaling Procedures

```bash
# Scale Copilot for high demand
kubectl scale deployment copilot-service --replicas=8 -n ai-system

# Scale NLP for processing backlog
kubectl scale deployment nlp-service --replicas=6 -n ai-system

# Scale Embedding for indexing workload
kubectl scale deployment embedding-service --replicas=6 -n ai-system
```

---

## 4. Maintenance Procedures

### 4.1 Qdrant Collection Maintenance
```bash
# Optimize collection indexes
curl -X POST "http://qdrant:6333/collections/agent_memory/points/optimize"
```

### 4.2 PostgreSQL Maintenance
```bash
# Vacuum analyze
VACUUM ANALYZE agent_executions;
VACUUM ANALYZE audit_log;
```

### 4.3 Agent Memory Cleanup
```bash
# Delete expired short-term memory
redis-cli -c KEYS "agent_state:*" | xargs -I {} redis-cli TTL {} | grep -c "^-1"
```

---

## 5. Emergency: Guardrail Bypass

If a guardrail is incorrectly blocking legitimate actions:

1. **DO NOT disable the guardrail service**
2. Identify the specific policy causing the block
3. Adjust policy threshold or condition
4. Test with policy simulator
5. Deploy updated policy
6. Monitor for correct behavior
