# Claude Code Remote (CCR) Operations

Internal tooling and operational runbooks for Claude Code Remote session management.

## Overview

Claude Code Remote (CCR) enables orchestrating multiple Claude sessions programmatically. This repository contains operational procedures, monitoring configs, and maintenance automation for CCR infrastructure.

## What is CCR?

CCR provides APIs for:
- **Session lifecycle management** - Create, monitor, and archive Claude sessions
- **Cross-session communication** - Send messages between sessions with priority control
- **Scheduled automation** - Trigger-based workflows with cron scheduling
- **Environment management** - Provision sandboxed execution environments

### Core Components

| Component | Description |
|-----------|-------------|
| Sessions | Individual Claude instances with isolated state |
| Environments | Sandboxed compute with repo access |
| Triggers | Scheduled or event-driven session spawning |
| Events | Audit log of session activity |

## Repository Structure

```
├── docs/
│   ├── architecture.md          # CCR system design
│   ├── api-reference.md         # MCP tool documentation
│   └── troubleshooting.md       # Common issues and fixes
├── ops/
│   ├── runbooks/                # Operational procedures
│   ├── monitoring/              # Alerting and dashboards
│   └── incident-response/       # Playbooks for outages
├── automation/
│   ├── maintenance/             # Scheduled cleanup jobs
│   ├── health-checks/           # Proactive monitoring
│   └── templates/               # PR/issue templates
├── configs/
│   ├── environments/            # Environment definitions
│   ├── triggers/                # Trigger configurations
│   └── thresholds/              # Alerting thresholds
└── scripts/
    ├── session-tools/           # CLI utilities
    └── migration/               # Version upgrade scripts
```

## Quick Start

### List Active Sessions
```bash
ccr list_sessions --json | jq '.sessions[] | {id, status, title, created_at}'
```

### Check Session Health
```bash
ccr list_events --session-id SESSION_ID --limit 20
```

### Archive Stale Sessions
```bash
ccr list_sessions --json | \
  jq -r '.sessions[] | select(.status == "IDLE") | .id' | \
  xargs -I {} ccr archive_session --session-id {}
```

### Create New Session
```bash
ccr create_session \
  --environment-id ENV_ID \
  --prompt "Your task description" \
  --permission-mode auto
```

## Common Operations

### Session Lifecycle

**States:**
- `PENDING` - Provisioning sandbox
- `RUNNING` - Actively processing
- `IDLE` - Waiting for input
- `ARCHIVED` - Terminated

**Typical flow:** PENDING → IDLE → RUNNING → IDLE → ... → ARCHIVED

### Environment Types

1. **Cloud** - General purpose, any repo access
2. **Repo-scoped** - Locked to specific repository
3. **Custom** - User-defined configurations

### Trigger Patterns

```javascript
// Every hour
{ cron: "0 * * * *" }

// Daily at midnight UTC
{ cron: "0 0 * * *" }

// Every 15 minutes during business hours
{ cron: "*/15 9-17 * * 1-5" }
```

## Operational Runbooks

### Stuck Sessions
Sessions stuck in PENDING > 10 minutes indicate provisioning failures.

**Diagnosis:**
1. Check `list_events` for "provision: started" without completion
2. Verify environment compatibility with source_url
3. Check infrastructure capacity

**Resolution:**
1. Archive stuck session
2. Recreate with explicit cloud environment_id
3. If recurring, escalate to infrastructure team

[Full runbook →](docs/troubleshooting.md#stuck-sessions)

### High Session Count
> 100 active sessions may indicate runaway triggers or resource leak.

**Diagnosis:**
1. Audit trigger configurations for aggressive schedules
2. Identify orphaned IDLE sessions
3. Check for missing cleanup automation

**Resolution:**
1. Disable problematic triggers
2. Archive sessions older than retention threshold
3. Implement automated cleanup

[Full runbook →](docs/troubleshooting.md#high-session-count)

### Environment Provisioning Failures
Sandbox allocation timeouts or capacity issues.

**Common causes:**
- Region capacity constraints
- Network configuration issues
- Resource quota exceeded

[Full runbook →](docs/troubleshooting.md#provisioning-failures)

## Monitoring

### Key Metrics

| Metric | Warning | Critical |
|--------|---------|----------|
| Active sessions | > 50 | > 100 |
| PENDING sessions | > 5 | > 10 |
| Avg provisioning time | > 60s | > 120s |
| Failed triggers (24h) | > 5 | > 20 |

### Alerts

Configured in [ops/monitoring/alerts.yaml](ops/monitoring/alerts.yaml):
- Stuck session detection
- Trigger failure rate
- Environment capacity
- Session count thresholds


## Maintenance

### Scheduled Tasks

| Task | Schedule | Description |
|------|----------|-------------|
| Stale cleanup | Daily 00:00 UTC | Archive IDLE sessions > 7 days |
| Trigger audit | Weekly Monday | Review trigger configurations |
| Capacity report | Monthly 1st | Environment utilization analysis |

### Manual Maintenance

Run ad-hoc maintenance:
```bash
./scripts/maintenance.sh --dry-run  # Preview changes
./scripts/maintenance.sh --execute  # Apply changes
```

Configuration: [configs/thresholds/maintenance.json](configs/thresholds/maintenance.json)

## Best Practices

### Session Management
- Always specify explicit `environment_id` for cross-repo work
- Use descriptive session titles for debugging
- Tag sessions with owner/purpose metadata
- Implement TTL-based cleanup

### Trigger Configuration
- Avoid `* * * * *` (every minute) schedules in production
- Use `send_later` for one-shot delayed tasks
- Test triggers in staging environment first
- Monitor trigger failure rates

### Environment Selection
- Use cloud environments for flexibility
- Repo-scoped for security isolation
- Cache environment IDs to avoid repeated lookups

### Error Handling
- Check session status before sending messages
- Handle "session not found" gracefully
- Implement retry logic with backoff
- Log all orchestration actions

## API Reference

### Session Operations
- `create_session` - Spawn new session
- `get_session` - Fetch session details
- `list_sessions` - Enumerate sessions
- `archive_session` - Terminate session
- `send_message` - Inject user message
- `interrupt_session` - Cancel current operation

### Environment Operations
- `list_environments` - Available sandboxes
- `add_repo` - Grant repo access
- `register_repo_root` - Configure workspace

### Trigger Operations
- `create_trigger` - Schedule automation
- `list_triggers` - View scheduled tasks
- `fire_trigger` - Manual execution
- `update_trigger` - Modify schedule
- `delete_trigger` - Remove automation

### Event Operations
- `list_events` - Session audit log


## Contributing

1. Create feature branch from `main`
2. Update relevant documentation
3. Test in staging environment
4. Open PR with runbook/procedure changes
5. Get review from ops team
