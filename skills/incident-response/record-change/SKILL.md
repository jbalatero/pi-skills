---
name: record-change
description: >
  Log infrastructure changes (deploy, config change, scaling, restart, migration) for future incident correlation.
  Use when the user wants to record a deploy, config change, scaling event, restart, database migration, or other infrastructure change.
disable-model-invocation: true
requires: incident-response MCP server
---

# Record Change

Log infrastructure changes so they get correlated during future incident investigations.

> **Prerequisite:** Requires the `incident-response` MCP server to be configured. Without it, this skill cannot function.

## How to Use

Call `record_change` by extracting these fields from the user's message:

| Field | Required | How to extract |
|-------|----------|----------------|
| `event_type` | yes | `deploy`, `config_change`, `scaling`, `restart`, `db_migration`, `dns_change`, `infra`, `rollback` |
| `service` | yes | The affected service name |
| `description` | yes | What changed — use the user's words |
| `source` | yes | `caprover_webhook`, `git_push`, `manual`, `docker_events` — default to `manual` |
| `commit_sha` | no | Git commit hash if mentioned |
| `image_tag` | no | Docker image tag if mentioned |
| `actor` | no | Who triggered it if mentioned |

## Mapping Informal Language

- "deployed" / "pushed" / "released" → `deploy`
- "changed config" / "updated env" / "updated connection string" → `config_change`
- "scaled up" / "added replicas" → `scaling`
- "restarted" / "bounced" → `restart`
- "ran migration" / "migrated" → `db_migration`
- "rolled back" / "reverted" → `rollback`

If the user doesn't specify the service or event type, ask.

## After Recording

Confirm what was recorded and remind the user that this change will now appear in future incident correlations.
