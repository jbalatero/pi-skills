---
name: session-retrospective
description: >
  Use at the end of a session or when asked to extract learnings — reviews conversation for patterns,
  gotchas, architecture decisions, operational knowledge, and tooling insights worth persisting to project rules.
disable-model-invocation: true
---

# Session Retrospective

## Overview

Extract learnings from the current session and persist them to project rule files. Every learning is presented for user approval before saving. Duplicates are skipped.

## Process

### 1. Read Existing Documentation

Read all project documentation (README, CONVENTIONS, rule files) to understand what's already documented. This prevents duplicates.

### 2. Scan Conversation for Learnings

Review the conversation for items worth persisting. Categories:

- **Domain patterns** — behavior, constraints, or relationships in the business domain
- **Pitfalls/gotchas** — things that broke or surprised, and the fix
- **Operational knowledge** — backup procedures, deployment quirks, infrastructure details
- **Architecture rules** — conventions, migration patterns, code organization decisions
- **Tooling/workflow** — linter configs, CI behavior, dev environment setup

### 3. Deduplicate

For each candidate learning, check if the same concept is already covered in an existing documentation/rule file. Skip if:
- The exact insight is already documented
- A more general version of the same insight exists
- The learning is already in the main project README or CONVENTIONS

If an existing entry is incomplete and the new learning adds meaningful detail, propose an **update** to the existing entry instead of a new one.

### 4. Present Numbered List

Present all non-duplicate learnings as a numbered list:

```
Learnings from this session:

1. [common-pitfalls.md] Remark linter errors when lint-staged passes ignored files — use --silently-ignore flag
2. [NEW: docker-gotchas.md] pnpm deploy --legacy doesn't create node_modules/.bin entries
3. [UPDATE: order-fitting-patterns.md] Add detail about install date calculator ignoring includes_hardware

Which items should I save? (all / none / comma-separated numbers)
```

Format: `[target-file.md]` for appends, `[NEW: filename.md]` for new files, `[UPDATE: filename.md]` for edits to existing entries.

### 5. Apply Approved Items

For each approved item:
- **Append:** Add a new `##` section to the target file
- **New file:** Create the file with a `#` heading and the learning as a `##` section
- **Update:** Edit the existing section in-place to incorporate the new detail

Use the same concise style as existing rule files — state the fact, the why, and the fix/implication. No narrative.

## What NOT to Save

- Session-specific context (current task details, in-progress work)
- Information already in project documentation
- Speculative conclusions from reading a single file
- One-off solutions unlikely to recur
- Standard practices well-documented elsewhere

## Writing Style

Match existing rule files:
- Bold heading summarizing the rule
- 1-3 sentences explaining the what and why
- Code example if it clarifies (inline, not verbose)
- No storytelling or narrative
