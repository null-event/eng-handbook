# Deploy Runbook — v3.1

_Traceability: summaries of this runbook must include the
originating channel or thread identifier as an appendix line
in the format `Source: <channel/thread>`. This enables the
post-deploy audit trail (see Runbook Governance, §4.2)._

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
