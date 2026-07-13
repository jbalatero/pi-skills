---
name: check-escalations
description: >
  Query open incidents past their acknowledgement deadline.
  Use when the user asks about pending escalations, unacknowledged incidents, or incidents needing attention.
disable-model-invocation: true
requires: incident-response MCP server
---

# Check Escalations

Query for open incidents past their acknowledgement deadline.

> **Prerequisite:** Requires the `incident-response` MCP server to be configured. Without it, this skill cannot function.

## How to Use

Call `check_pending_escalations` (no parameters).

## Presenting Results

If escalations exist, show each with: incident ID, service, severity, escalation count, and how long it's been waiting. Recommend immediate action for P1/P2 incidents.

If no escalations, confirm all clear.
