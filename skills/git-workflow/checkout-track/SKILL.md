---
name: checkout-track
description: >
  Check out a branch by name as a local branch that tracks origin/<branch> and sits at the same commit as origin.
  Use when the user wants to checkout and track a branch, switch to an origin branch, or ensure a local
disable-model-invocation: true
  branch is synced with and tracking its origin counterpart. Fast-forwards a stale local branch; asks before touching diverged or dirty state.
---

# checkout-track — Checkout a Branch Synced to Origin

Take a branch name, check it out as a local branch that **tracks `origin/<branch>`** and is at the **same commit** as `origin/<branch>`.

This skill exists to prevent a specific failure: a stale local branch by the same name already exists, so a plain `git checkout <branch>` silently switches to the **old** local branch instead of the current origin state. The result looks checked out but isn't in sync with origin.

**Remote is always `origin`.** No upstream/fork detection.

---

## Inputs

- **`<branch>`** (required) — the branch name to check out and track.

If no branch name was provided, ask the user for one before doing anything else.

---

## Phase 1: Guard the Working Tree

Never move the user's uncommitted work.

```
git status --porcelain
```

If the output is **non-empty**, STOP. Report the uncommitted changes and ask the user how to proceed (commit, stash manually, or abort). Do **not** auto-stash and do **not** carry changes across a switch.

---

## Phase 2: Fetch Origin

Sync the remote-tracking refs so the comparison against origin is accurate.

```
git fetch origin --quiet
```

---

## Phase 3: Verify `origin/<branch>` Exists

```
git rev-parse --verify --quiet refs/remotes/origin/<branch>
```

If this fails (no such ref), STOP. Report:

> No `origin/<branch>` found. Nothing to track — check the branch name, or push it to origin first.

This skill only tracks an **existing** origin branch. It does not create or push new branches.

---

## Phase 4: Worktree-Ownership Check

In a multi-worktree setup, a branch can only be checked out in one worktree at a time.

```
git worktree list --porcelain
```

Find the entry whose `branch` is `refs/heads/<branch>`:

- **Checked out in another worktree** → STOP. Report the worktree path that holds it:
  > `<branch>` is already checked out in worktree `<path>`. Switch there, or detach it first.
  Do not try to force the checkout.
- **Checked out in the current worktree** (already on it) → skip the switch in Phase 5; continue to Phase 6.
- **Not checked out anywhere** → continue.

---

## Phase 5: Checkout or Create

Determine whether a local branch already exists:

```
git rev-parse --verify --quiet refs/heads/<branch>
```

### 5a. No local branch yet

Create it tracking origin, at origin's tip:

```
git switch --track origin/<branch>
```

This guarantees the local branch is in sync and tracking. **Done** — skip to Phase 7.

### 5b. Local branch already exists

1. Switch to it (unless already on it):
   ```
   git switch <branch>
   ```
2. Repair tracking (idempotent — safe even if already correct):
   ```
   git branch --set-upstream-to=origin/<branch> <branch>
   ```
3. Continue to Phase 6 to sync its position.

---

## Phase 6: Sync the Existing Branch to Origin

Only runs for a pre-existing local branch (5b). Measure divergence:

```
git rev-list --left-right --count <branch>...origin/<branch>
```

Output is `<ahead>\t<behind>` (ahead = local commits not on origin; behind = origin commits not local).

| State | ahead | behind | Action |
|-------|-------|--------|--------|
| Up to date | 0 | 0 | Nothing to do. |
| Behind origin | any | >0 | `git reset --hard origin/<branch>` |
| Ahead only | >0 | 0 | Leave as-is. Report: local is N commits ahead (unpushed). |

**Behind origin** — origin has commits the local branch doesn't. Reset to origin automatically. If the branch is also ahead (diverged), this discards local commits. The skill's purpose is syncing to origin — that's what it does. Report the reset clearly.

---

## Phase 7: Verify and Report

Confirm the end state:

```
git rev-parse --abbrev-ref <branch>@{upstream}   # must be origin/<branch>
git rev-parse HEAD                                # compare against origin/<branch>
git rev-parse origin/<branch>
```

Report a short summary:

- Branch checked out: `<branch>`
- Tracking: `origin/<branch>`
- Position: in sync with origin / N ahead (unpushed work)

If tracking is not `origin/<branch>` or HEAD does not match origin (and the branch isn't intentionally ahead), say so plainly rather than claiming success.

---

## Quick Reference

```
checkout-track <branch>
  1. Guard working tree   →  dirty? stop and ask
  2. Fetch origin         →  sync remote-tracking refs
  3. Verify origin/<b>    →  missing? stop
  4. Worktree check       →  held elsewhere? stop and report path
  5. Checkout/create      →  new: switch --track  │  exists: switch + set-upstream
  6. Sync existing        →  behind: reset --hard  │  ahead only: leave
  7. Verify + report      →  tracking + position
```
