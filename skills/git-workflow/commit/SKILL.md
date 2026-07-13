---
name: commit
description: >
  Git commit with conventional format, diff analysis, and split suggestions.
  Use when the user wants to commit changes, create a git commit, or stage and commit work.
disable-model-invocation: true
---

# Commit — Conventional Commits

## Process

1. Check staged files, commit only staged files if any exist
2. Analyze diff for multiple logical changes
3. Suggest splitting if needed
4. Create commit with conventional format
5. Pre-commit hooks run automatically

## Split Criteria

Different concerns | Mixed types | File patterns | Large changes

## Commit Format

Follow Conventional Commits 1.0.0 with this format in plain-text only and without open/close quotes or backticks:

```
<type>[optional scope]: <description>

[optional body]

[optional footer]
```

Optional body lines must not be longer than 100 characters.

### Allowed Types

- **feat**: New features
- **fix**: Bug fixes
- **docs**: Documentation updates
- **style**: Non-functional style changes
- **refactor**: Code restructuring (no fixes/features)
- **perf**: Performance improvements
- **test**: Adding/fixing tests
- **build**: Build system/dependency changes
- **ci**: CI/CD updates
- **chore**: Non-source changes (e.g., tooling)

### Description Rules

- Imperative, present tense ("add" not "added")
- No capitalization or period
- ≤72 characters, clear and specific

### Scope Usage

- Lowercase, consistent, always snake_case
- Match project directory structure

### Breaking Changes

- Append `!` after type/scope (feat!: ...)
- Include `BREAKING CHANGE:` in footer with migration details

## Notes

- Pre-commit hooks handle checks
- Only commit staged files if any exist
- Analyze diff for splitting suggestions
- **NEVER add Claude signature or agent signature to commits**
