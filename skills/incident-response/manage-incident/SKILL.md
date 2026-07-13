---
name: manage-incident
description: >
  Handle incident lifecycle: acknowledge, snooze, mark as investigating, or resolve an incident.
  Use when the user wants to ack, investigate, snooze, or resolve an incident.
disable-model-invocation: true
requires: incident-response MCP server
---

# Manage Incident

Handle incident lifecycle — acknowledge, investigate, snooze, or resolve.

> **Prerequisite:** Requires the `incident-response` MCP server to be configured. Without it, this skill cannot function.

## Routing

### Acknowledge / Snooze / Investigating

Call `acknowledge_incident` with:
- `incident_id`: UUID from user message
- `action`: `ack`, `investigating`, or `snooze`
- `snooze_minutes`: duration if snoozing (5–480 min). Map "30 minutes" → 30, "1 hour" → 60
- `notes`: any context the user provides

Map user intent:
- "acknowledge" / "ack" → `action: "ack"`
- "investigating" / "looking into it" → `action: "investigating"`
- "snooze" / "delay" / "wait" → `action: "snooze"` (ask for duration if not provided)

### Resolve

Call `resolve_incident` with:
- `incident_id`: UUID
- `root_cause_category`: one of `deploy_regression`, `config_error`, `resource_exhaustion`, `dependency_failure`, `network`, `database`, `health_check`, `scaling`, `security`, `unknown`, `other`
- `root_cause_detail`: what specifically broke (extract from user's explanation)
- `fix_action`: what was done to fix it
- `fix_type`: `rollback`, `config_fix`, `restart`, `scale_up`, `code_fix`, `dependency_update`, `infra_change`, `manual_intervention`, `self_healed`, `other`
- `prevention_notes`: optional, how to prevent recurrence

Extract all fields from the user's natural language description. If critical fields are missing (incident_id, root cause, fix), ask.

Remind the user that resolutions feed the knowledge base — future incidents with the same pattern will surface this fix as a recommendation.
