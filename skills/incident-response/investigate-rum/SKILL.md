---
name: investigate-rum
description: >
  Trace frontend RUM errors, views, resources, or requests through the full stack — across OpenObserve RUM streams and MySQL event store.
  Use when the user wants to trace a frontend RUM event through the backend, debug a RUM error, or follow a request chain.
disable-model-invocation: true
requires: incident-response MCP server
---

# Investigate RUM Data

You are an expert SRE. The user wants to trace a frontend RUM event through the full stack. Call `investigate_rumdata` with the appropriate entry point, then synthesize the chain into a readable report.

> **Prerequisite:** Requires the `incident-response` MCP server to be configured. Without it, this skill cannot function.

## Entry Point Selection

Pick exactly one entry point based on what the user provides:

| User provides | Parameter |
|---------------|-----------|
| RUM error ID | `error_id` |
| RUM view / page ID | `view_id` |
| RUM resource ID | `resource_id` |
| Backend API resource UUID | `api_resource_id` |
| Distributed trace ID | `trace_id` |
| Backend request ID (rid) | `request_id` |

If the user provides a raw identifier without labelling it, infer the type from context or ask one clarifying question.

Use `time_window_minutes` (default: 60) if the user specifies a narrower or wider time window.

## How the Chain Works

The tool chases correlation IDs downward through streams automatically:

```
error_id
  └─→ view_id (_rumdata)
        └─→ resource_id(s) (_rumdata, type=resource) — fans out
              └─→ api_resource_id
                    └─→ main stream (HttpAccess/HttpMutation) → extracts rid + trace_id
                          ├─→ trace_id → main stream (all spans in trace)  [terminal]
                          └─→ rid     → MySQL evtstore_events WHERE request_id = rid  [terminal]
```

Each step queries OpenObserve (or MySQL at the terminal steps), collects raw records, extracts the next IDs to chase, and reports a **dead end** if it cannot continue. The chain fans out when a view has multiple resources — each resource gets its own sub-chain. `trace_id` and `request_id` used as entry points skip directly to their respective terminal step.

## After Receiving Results

Synthesize the `steps`, `trace_ids`, `trace_urls`, and `dead_ends` into a report.

### Synthesis Guidelines

- Build the chain summary in traversal order — show each step, which stream was queried, how many records were found, and what IDs were extracted
- For dead ends, explain the likely cause (see common patterns below)
- For key findings, focus on errors, slow resources, or missing backend traces — not just what was found but what it implies
- If the chain fans out across multiple resources, prioritize resources with error status codes or high duration

### Common Patterns

- **No trace URL generated** — the resource had no `api_resource_id`; the request may not have reached the backend, or the RUM instrumentation is missing the correlation header
- **Dead end at api_resource_id** — no matching backend log; check if the service was restarting or if the request was dropped at the load balancer
- **Many resources for one view** — focus on resources with error status codes or high duration first
- **Chain stops at request_id** — check MySQL `evtstore_events` records for command/event details

### Report Format

    ## RUM Trace: [entry point type] `<value>`

    ### Chain Summary
    | Step | Stream | Records | Extracted |
    |------|--------|---------|-----------|
    | error_id → view_id | _rumdata | N | view_id: ... |
    | ... | ... | ... | ... |

    ### Trace Links
    - <trace_url 1>
    - ...

    ### Dead Ends
    - <dead_end description>

    ### Key Findings
    <What the trace reveals — errors seen, slow resources, missing backend traces, etc.>

    ### Recommended Next Steps
    1. <action>
    2. <action>
