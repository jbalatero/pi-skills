---
name: generate-changelog
description: >
  Generate a changelog entry from git history between two refs.
  Use when the user wants to generate changelog, create release notes, or document what changed between versions.
disable-model-invocation: true
---

# Generate Changelog

Generate a changelog entry by analyzing git history between two refs. Works standalone or as part of a release workflow.

## Input

Extract the range or options from the user's prompt. If none specified, uses sensible defaults:

- `v1.2.0..HEAD` — changelog for commits since tag v1.2.0
- `main..staging` — changelog for what's on staging but not main
- `--since 2026-03-15` — changelog for commits since a date
- (empty) — changelog for commits between latest tag and HEAD

## Configuration

Read the `## Git Workflow Config` section from the project documentation for optional config:
- `changelogPath` — output directory (e.g., `docs/changelogs/`). If not set, print to stdout
- `planDocsPath` — if set, checks for design docs to enrich changelog content

## 1. Determine Range

- If the user specified a range (e.g., `v1.0..HEAD`), use it
- If the user specified `--since <date>`, use `git log --since=<date>`
- If empty, find the latest tag (`git describe --tags --abbrev=0`) and use `<latest-tag>..HEAD`
- If no tags exist, use all commits on the current branch not on main

Validate the range has commits. If empty, stop and report "no changes found in range."

## 2. Gather Source Material

For each commit/PR in the range:

1. Extract PR numbers from commit messages (e.g., `(#123)`, `Merge pull request #123`)
2. For each PR found, gather context in this priority order:
   - If `planDocsPath` is configured, check for a design doc matching the PR number or feature topic
   - Analyze the actual code diffs to understand what was implemented
   - Check `gh pr view <number>` for the PR title and body
3. For commits without PR numbers, analyze the commit message and diff directly

Group changes by type using conventional commit prefixes (feat, fix, refactor, etc.).

## 3. Generate Changelog

**If `changelogPath` is configured:**

- Determine the next filename:
  - Glob `<changelogPath>/YYYY-MM-DD.*.md` for today's date
  - Next number = count of matches + 1, zero-padded to 3 digits
  - Filename: `<changelogPath>/YYYY-MM-DD.NNN.md`
- Write the changelog file with this structure:
  - `# Release YYYY-MM-DD.NNN`
  - `> summary line` (under 200 chars, no double quotes) — plain language understandable by non-technical people
  - Reference lines as additional `>` blockquotes, one per PR (if PR numbers available)
  - One `## feat/fix(scope): Description (#NNN)` section per change with overview, key changes, and notable decisions
  - `## Rollback` section at the end

**If no `changelogPath`:**

- Print the changelog content directly to the conversation using the same structure above
- Ask the user if they want to save it to a file

## 4. Offer to Commit (if file was written)

Ask the user if they want to commit the changelog. If yes:

```
docs: add changelog for YYYY-MM-DD.NNN release
```

Stage only the changelog file.

## Notes

- Skip commits that are exclusively non-functional (merge commits with no code changes, empty commits)
- Write descriptions in past tense for changelog entries ("Added X" not "Add X") — changelogs describe what happened, not instructions
- Keep each section concise but specific enough that someone unfamiliar with the codebase understands the impact
- If the range contains too many commits (50+), summarize by category rather than listing every change individually
