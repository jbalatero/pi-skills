---
name: push-to-target
description: >
  Push commits to target branch via cherry-pick when target cannot be checked out locally (worktree conflict).
  Use when you need to get changes to staging/target but git checkout fails with "already used by worktree".
disable-model-invocation: true
---

# Push to Target Branch

Push one or more commits to the target branch without checking it out locally. Use when `git checkout <target>` fails with `fatal: '<target>' is already used by worktree`.

## Configuration

Read the `## Git Workflow Config` section from the project documentation for configuration. If not present, use defaults: target branch = `staging`.

## Determine What to Push

1. **Uncommitted changes** — use the `commit` skill to create the commit, then record the hash
2. **Existing commit(s)** — the user specifies a hash or range, or confirms the latest commit(s) on the current branch

Record the commit hash(es) to cherry-pick.

## Pre-flight

```bash
git fetch origin <target>
git log origin/<target> -1 --oneline
```

Show the user the latest target branch commit so they can confirm this is the right target.

## Cherry-pick Workflow

1. **Stash any uncommitted changes** (if working tree is dirty):
   ```bash
   git stash --include-untracked
   ```
2. **Create a temp branch from origin/<target>**:
   ```bash
   git checkout -B temp-target-push origin/<target>
   ```
3. **Cherry-pick the commit(s)**:
   ```bash
   git cherry-pick <hash>
   ```
   If conflicts occur, **stop and ask the user** — do not force-resolve.
4. **Push to target branch**:
   ```bash
   git push origin HEAD:<target>
   ```
5. **Return to the original branch and clean up**:
   ```bash
   git checkout <original-branch>
   git branch -D temp-target-push
   ```
6. **Restore stashed changes** (if step 1 stashed):
   ```bash
   git stash pop
   ```
7. **Report** the commit hash(es) pushed and confirm they landed on `origin/<target>`
