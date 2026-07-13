---
name: gather-rules
description: >
  Explore codebase, discover conventions and patterns, and save as path-scoped rules.
  Use when the user wants to extract project conventions, document codebase patterns, or create rules from the code.
disable-model-invocation: true
---

# Gather Rules

Explore the codebase to discover conventions, architectural patterns, and implicit rules that are not yet captured in project documentation. Present findings for approval, then save as properly scoped rule files.

## Input

Extract a focus area from the user's prompt. If none specified, perform a broad sweep. Examples:

- `API patterns` — controllers, services, DTOs, route registration
- `testing conventions` — test framework patterns, mock factories, fixtures
- `state management` — store patterns, data flow, caching
- `database` — ORM conventions, migrations, query patterns
- `frontend` — component patterns, styling, routing
- `error handling` — error classes, middleware, logging
- `CI/CD` — build pipeline, deployment patterns
- `authentication` — auth middleware, permission patterns

## 1. Read Existing Documentation

- Glob `*.md` files in the project root (README.md, CONVENTIONS.md, CONTRIBUTING.md, etc.)
- Build an inventory of already-covered topics so you never create duplicates
- Check for any rule/pattern files already present

## 2. Explore the Codebase

- Analyze the codebase broadly, or narrowed by focus area
- Look for patterns that aren't obvious from reading the code once — things that would cause bugs or wasted time if not known upfront
- **What to look for** (general pattern categories):
  - Recurring code patterns (naming, structure, error handling)
  - Architectural conventions (module boundaries, data flow)
  - Configuration conventions (env vars, config files, build tools)
  - Security and access control patterns
  - API design patterns (request/response, serialization, validation)
  - Database and ORM conventions
  - Deployment and infrastructure patterns
  - Testing patterns and conventions
  - Frontend component and state management patterns
- Skip anything already covered by existing documentation
- Aim for at least 5 rules, up to 15, depending on focus area

## 3. Present Findings

Show the user a numbered table with these columns:

| # | Proposed Filename | Paths Scope | Summary |
|---|-------------------|-------------|---------|

For each proposed rule, also show the full content that would be written to the file.

**Path scoping guidelines:**
- Rules that apply only when working with specific directories get path scoping
- Rules that are universal (git workflow, infra, architectural decisions) have NO path scoping
- Cross-cutting rules that span multiple areas list all relevant path patterns
- Use glob patterns matching the project's directory structure

**Rule quality criteria:**
- Each rule should capture something non-obvious that would cause a bug, wasted effort, or style violation if not known
- Avoid restating what's already in project docs — rules should capture patterns learned from the code itself
- Include concrete code examples where they prevent misuse
- Name files with kebab-case matching the topic (e.g., `sequelize-model-conventions.md`)

**STOP HERE.** Do not write any files until the user explicitly approves. The user may:
- Approve all
- Remove items by number
- Request edits to specific items
- Ask for more exploration in a specific area

## 4. Write Approved Rules

For each approved rule, write to `.claude/rules/<filename>.md` with this format (or equivalent project rule directory):

```markdown
# <Rule Title>

## <Section heading>

<Content — concise, actionable, pattern-focused>
```

- Omit path scoping if universal rules
- Use clear markdown with code examples where they add clarity
- Keep each rule focused on one topic — split broad findings into separate files

## 5. Offer to Commit

After writing all files, ask the user if they want to commit. If yes, use conventional commit format:

```
docs: add rules for <brief summary of topics>
```

Stage only the rule files that were created or modified.
