---
name: squash-merge-pr
description: Use when an open PR needs to be squash merged into the target branch — after review approval, when merging feature work, or when told to merge a PR
disable-model-invocation: true
---

# Squash Merge PR to Target Branch

Squash merge an existing PR into the target branch and optionally annotate the plan file with the PR number.

## Configuration

Read the `## Git Workflow Config` section from the project documentation for configuration. The target branch is always the PR's `baseRefName` (fetched in step 1). For fallback when no PR number is provided yet:

1. `targetBranch` from `## Git Workflow Config` (if present)
2. Fall back to `staging`

If `planDocsPath` is not configured, default to none.

## Inputs

- **PR number** (required): The PR to merge. If not provided, detect from current branch: `gh pr view --json number --jq '.number'`

## Steps

### 1. Validate PR State

```bash
gh pr view <NUMBER> --json state,baseRefName,mergeable,headRefName
```

- Confirm `state` is `OPEN`
- Set target branch to `baseRefName` from the PR (this is the authoritative target)
- Confirm `mergeable` is `MERGEABLE`
- If any check fails, stop and report

### 2. Squash Merge

```bash
gh pr merge <NUMBER> --squash
```

If merge fails, check `gh pr checks <NUMBER>` and report.

### 3. Switch to Target Branch

```bash
git checkout <target> && git pull origin <target>
```

If `git checkout <target>` fails due to worktree conflict (`fatal: '<target>' is already used by worktree`), run `git fetch origin <target>` instead — the remote merge already succeeded.

### 4. Update Plan File with PR Number (if planDocsPath configured)

1. Extract feature topic from the PR's branch name:
   `feature/abandoned-cart` → `abandoned-cart`

2. Search for matching plan file:
   ```bash
   ls <planDocsPath>/*{topic}*.md
   ```

3. If a matching plan file is found:
   - Read the file, find the first `#` heading
   - Append ` (#NNN)` to that heading
   - Stage, commit: `docs: link PR #NNN to plan file`
   - Push to target branch

4. If no matching plan file or planDocsPath not configured, skip and inform the user.

### 5. Report

Summarize: PR number, merge status, plan file updated (if applicable).
