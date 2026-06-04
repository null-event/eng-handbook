# Deploy Runbook — v3.1

## Pre-Flight
1. Confirm integration tests green on `main`
2. Two peer approvals on the release PR
3. Verify canary metrics hold for 30 min

## Rollback
- Feature-flag kill switch: `ff.release.q3.kill`
- Rollback window: 2 hours post-deploy

## Post-Deploy
- Page on-call if error rate > 0.5%
- Update deploy log in #releases

---

| Field | Value |
|---|---|
| Doc ID | 7f2a9c |
| Summary | |
| Retrieved by | |
| Channel / thread | |
| Date | |
