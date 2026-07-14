---
name: deploy
description: >
  Full target-to-production deployment: changelog generation, env var diff check, pre-deploy script, rebase to main.
  Use when the user wants to deploy, ship to production, release, or push to main from a target/staging branch.
disable-model-invocation: true
---

# Deploy to Production

Automates the full target branch → main deployment pipeline. Run this from the target branch with a clean working tree.

## Configuration

Determine the target branch using this priority:
1. Branch specified by the user in their prompt
2. `targetBranch` from the `## Git Workflow Config` section in project documentation
3. Falls back to `staging` if neither is provided

Read the `## Git Workflow Config` section from the project documentation for additional configuration. Optional config:
- `targetBranch` — default target branch for deployments
- `changelogPath` — if set, generates changelogs (e.g., `docs/changelogs/`)
- `preDeployScript` — if set, runs before deploy (e.g., `./pre-deploy.sh`)
- `planDocsPath` — if set, checks for plan docs to source changelog content
- `rollbackInstructions` — if set, used as the content of the `## Rollback` section in the changelog

## Steps

### 1. Validate State

- Confirm current branch is the target branch
- Confirm working tree is clean (no uncommitted changes)
- Confirm target is ahead of main (`git log main..<target> --oneline`)
- If any check fails, stop and report what's wrong

### 2. Generate Changelog (if changelogPath configured)

- Get PRs merged since the last deploy: extract PR numbers from `git log main..<target> --oneline`
- For each PR, gather source material in this priority order:
  1. If `planDocsPath` is configured, check for a design doc matching the PR number or feature topic
  2. **If no design doc, analyze the actual code diffs** for that PR's commits to understand what was implemented
  3. Optionally check `gh pr view <number>` for hints from the PR title and body
- Determine the next changelog filename:
  - Glob `<changelogPath>/YYYY-MM-DD.*.md` for today's date to find existing files
  - Next number = count of matches + 1, zero-padded to 3 digits (e.g., `.001`, `.002`, `.003`)
  - Filename: `<changelogPath>/YYYY-MM-DD.NNN.md`
- Write the changelog file:
  - `# Release YYYY-MM-DD.NNN`
  - `> summary line` (under 200 chars, no double quotes) summarizing all PRs — plain language understandable by non-technical people
  - Reference lines as additional `>` blockquotes after the summary, one per PR (if PR numbers available)
  - One `## feat/fix(scope): Description (#NNN)` section per PR with overview, key changes, and notable decisions
  - Always add a `## Rollback` section at the end — if `rollbackInstructions` is configured in `## Git Workflow Config`, use that as the section content; otherwise write generic rollback steps based on the changes
- **Always generate a changelog** if there are code changes in the diff. Only skip if the diff contains exclusively non-functional changes.

### 3. Check for ENV Var Changes

- Scan the diff (`git diff main..<target>`) for any changes to ENV var files or references. Look for:
  - Changes to `.env.example`, `.env.sample`, or similar template files
  - New, modified, or removed keys in any `.env*` file tracked in git
  - Code changes that add, rename, or remove references to environment variables (e.g., `process.env.NEW_VAR`, `ENV['NEW_VAR']`, `os.environ.get('NEW_VAR')`)
- If any ENV var changes are detected:
  - List each change clearly: added keys, removed keys, renamed keys
  - Display this message to the user:

    > **Action required before deploy:**
    > The following ENV var changes were detected. Please apply them to the live server before continuing:
    > [list changes]
    > Once done, reply "done" (or similar) to proceed with the deployment.

  - **Wait for the user to confirm** before continuing to the next step
  - Do not proceed automatically — this step requires explicit user confirmation
- If no ENV var changes are detected, continue to the next step without pausing

### 4. Run Pre-Deploy (if preDeployScript configured)

- Execute the configured pre-deploy script
- If it fails, stop and report the error

### 5. Commit

- Stage the changelog file (if generated) and any lockfile changes from pre-deploy
- If both: commit with message `build: pre-deploy and changelog for YYYY-MM-DD.NNN release`
- If only lockfile changes: `build: pre-deploy`
- If only changelog: `docs: add changelog for YYYY-MM-DD.NNN release`
- Skip if there is nothing to commit

### 6. Fast-Forward Main

- `git fetch origin main` — get latest main from origin
- Verify fast-forward: confirm `origin/main` is an ancestor of `<target>` (`git merge-base --is-ancestor origin/main <target>`)
- If not fast-forwardable, stop and report the issue — do not force-push
- This avoids worktree conflicts since it never checks out main

### 7. Push to Main and Target

- `git push origin <target>:main` — push target branch to main
- If this fails (e.g., branch protection), stop and report the error

### 8. Push Target to Origin

- `git push origin <target>` — push the changelog commit (if any) to origin's target branch
- Report success with a summary of what was deployed (PR numbers, changelog file path if applicable)

## Error Handling

- Stop at the first failure and report clearly what went wrong
- Never force-push
- If step 6 fails (main is not an ancestor of target), report that a fast-forward is not possible and stop
- Always leave the repo in a clean state on the branch the user started from
