# Rollback — Vision Moderation

## When to roll back

Trigger rollback if **any** of the following fire and do not self-heal in 5 min:

- [ ] `AvailabilityBurnFast` alert active (error rate > 7.2% over 1 h)
- [ ] `LatencyP99High` alert active (p99 > 1 s for 5 min)
- [ ] `ModelVersionMismatch` alert active (replica on wrong model version)
- [ ] Human-review queue spike > 50% above baseline (quality proxy)

---

## How to roll back

```bash
# 1. Find the previous good SHA
git log --oneline -5

# 2a. Preferred — trigger via GitHub CLI
gh workflow run deploy-model.yml \
  -f action=rollback \
  -f version=<previous-sha>

# 2b. Fallback — run the script directly
./scripts/rollback.sh production <previous-sha>
```

---

## What to verify (all must turn green)

- [ ] `AvailabilityBurnFast` resolves within 5 min
- [ ] `LatencyP99High` resolves within 5 min
- [ ] `ModelVersionMismatch` resolves within 5 min
- [ ] Grafana › Vision Moderation: error rate < 0.5%, p99 < 1 s
- [ ] `model_version` label on all pods matches the rolled-back SHA

---

## Who to notify

| Event | Who | Channel |
|---|---|---|
| Rollback started | On-call lead | `#ml-platform-incidents` |
| Rollback complete | Team + stakeholders | `#ml-platform-incidents` + Linear incident ticket |
| Root cause unclear after 30 min | ML team + backend on-call | Direct PagerDuty page |

---

## What NOT to do

- **Do not** re-deploy the new version until root cause is confirmed
- **Do not** manually patch the running container — use `rollback.sh`
- **Do not** close the Linear incident ticket until production alerts are green
- **Do not** skip the Trivy scan gate when re-deploying the fix

---

## When to roll forward

1. Root cause identified and fixed in a new commit
2. New image passes Trivy scan (0 HIGH/CRITICAL CVEs)
3. All tests pass; smoke test passes in staging
4. Required approver signs off in the GitHub production environment gate
