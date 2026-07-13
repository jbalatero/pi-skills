# Pi Skills — jbalatero

Deterministic `/skill:name` commands converted from [jbalatero/claude-plugins](https://github.com/jbalatero/claude-plugins) for use with [Pi](https://github.com/badlogic/pi) coding agent.

## Install

### Option A: `pi install` (recommended)

```bash
pi install git:github.com/jbalatero/pi-skills
```

Then add to `~/.pi/agent/settings.json`:

```json
{
  "enableSkillCommands": true
}
```

### Option B: Manual clone

```bash
git clone https://github.com/jbalatero/pi-skills.git ~/pi-skills
```

Then add to `~/.pi/agent/settings.json`:

```json
{
  "skills": ["~/pi-skills/skills"],
  "enableSkillCommands": true
}
```

## Commands

### git-workflow

| Command | Description |
|---------|-------------|
| `/skill:commit` | Conventional commits with diff analysis and split suggestions |
| `/skill:push-to-target` | Push commits to target via cherry-pick (worktree-safe) |
| `/skill:create-pr-merge` | Create PR, squash merge, annotate plan file |
| `/skill:rebase-target` | Rebase with intelligent conflict resolution |
| `/skill:deploy` | Full target→main deployment: changelog, env diff, pre-deploy, rebase |
| `/skill:checkout-track` | Checkout branch synced to origin |
| `/skill:create-pr` | Create PR with templates and conventional titles |
| `/skill:git-good` | Safety guardrails for destructive git operations |
| `/skill:prune` | Interactive stale branch cleanup |
| `/skill:squash-merge-pr` | Squash merge existing PR + plan file annotation |

### dev-process

| Command | Description |
|---------|-------------|
| `/skill:audit-dependencies` | Security vulnerability & outdated package audit |
| `/skill:gather-rules` | Discover codebase patterns, save as rules |
| `/skill:generate-changelog` | Generate changelog from git history |
| `/skill:pre-implementation-review` | Surface assumptions/edge cases before coding |
| `/skill:review-github-actions` | actionlint + security review for GHA workflows |
| `/skill:session-retrospective` | Extract learnings → persist to rules |

### incident-response *(requires MCP server)*

| Command | Description |
|---------|-------------|
| `/skill:investigate` | Investigate incidents, healthcheck failures |
| `/skill:investigate-rum` | Trace frontend RUM errors full-stack |
| `/skill:manage-incident` | Lifecycle: ack, snooze, investigate, resolve |
| `/skill:check-escalations` | Query incidents past ack deadline |
| `/skill:record-change` | Log infra changes for incident correlation |

## Configuration (git-workflow skills)

Add a `## Git Workflow Config` section to project README/CONVENTIONS:

```markdown
## Git Workflow Config
- Target branch: `staging`
- Plan docs path: `docs/plans/`
- Changelog path: `docs/changelogs/`
- Pre-deploy script: `./pre-deploy.sh`
```

All values optional (default target = `staging`).

## Credits

Converted from [jbalatero/claude-plugins](https://github.com/jbalatero/claude-plugins).
