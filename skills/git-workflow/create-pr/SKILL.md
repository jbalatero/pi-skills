---
name: create-pr
description: >
  Create a pull request, merge request, or change request with proper formatting and content guidelines.
  Use when the user wants to create, open, or submit a PR, MR, or CR — including after committing changes.
disable-model-invocation: true
---

# Create Pull Request

## Configuration

Read the `## Git Workflow Config` section from the project documentation for configuration. Determine the target branch using this priority:

1. `targetBranch` from `## Git Workflow Config` (if present)
2. GitHub repo default branch: `gh repo view --json defaultBranchRef --jq '.defaultBranchRef.name'`
3. Fall back to `staging`

## Context

- Remote URL: get via `git remote get-url origin`
- Recent commits: get via `git log --oneline -10`

## Title

- Check the log in the context above to determine the repo's commit style:
  - **subject** (default): `${subject}: ${summary}` (e.g., `api: add timeout to request`)
  - **conventional**: `${type}: ${summary}` (e.g., `fix: add timeout to request`)
- Keep under 50 characters, max 100
- Use imperative mood, lowercase except proper nouns

## Body

- Start with 1-3 sentences summarizing the change (no preceding header). Use active voice and direct language
  - Good: "Adds retry logic to the HTTP client and limits maximum attempts to 3."
  - Bad: "Retry logic is added to the HTTP client and maximum attempts are limited to 3."
- If the PR is directly motivated by an issue, reference it at the end of the opening summary (`Closes #N`, `Fixes #N`, or just `#N` if not closing). Related issues mentioned for context belong in a `## References` section. Never drop issue refs at the bottom of the body without a section
- **Wrap all code identifiers with backticks**: function names, class names, file paths, endpoints, status codes, etc.
- Use `##` sections for larger changes. See [`sections.md`](sections.md) for detailed guidance on:
  - `## Issue` - Root cause analysis and issue linking
  - `## Changes` - High-level description of changes
  - `## Testing` - Test coverage insights
  - `## References` - Related links and issues

## Template

If a PR template exists at `.github/PULL_REQUEST_TEMPLATE.md` or `.github/pull_request_template.md`, read it and follow its structure instead of the default body format:

- Preserve all template sections, even if some are left empty
- Leave checklists (checkbox items) untouched for the user to complete manually
- Remove HTML comments (`<!-- ... -->`) that serve as placeholder instructions
- Map skill-generated content into corresponding template sections:
  - Description/summary sections: use the opening summary sentences
  - Changes/what sections: use `## Changes` content from `sections.md`
  - Testing/verification sections: use `## Testing` content from `sections.md`
  - Issue/references sections: use `## Issue` and `## References` content
- For template sections with no skill equivalent (e.g., type-of-change dropdowns), fill them based on the diff context
- Within each template section, follow the style rules from `sections.md`
- If the template has a free-form description section, place the summary sentences there and add skill subsections within it as needed

When no template is detected, use the default body format from the Body section above.

## Issue Handling

When an issue is referenced:

- **ONLY reference the issue** in the PR body (e.g., `Closes #123`, `Fixes #456`)
- **NEVER modify the issue directly** - no comments, labels, milestones, or assignees

## Workflow

1. **Read config**: Determine target branch per the Configuration section above (config → GitHub default → `staging`).
1. **Branch validation**: If currently on the target branch, stop and ask the user to switch to a feature branch first.
1. Stage changes if not already staged: `git add .`
1. Commit if there are no commits yet on the branch. Follow the same format for the commit message as for the pull request title (conventional or subject-oriented based on repo standard): `git commit -m "..."`
1. Push the branch to remote: `git push -u origin HEAD`
1. Create the PR/MR:
   - Write the body to a temp file first (e.g., `tmp/pr-body-<branch>.md`)
   - Include the branch name in the filename to avoid conflicts with concurrent agents
   - **GitHub**: `gh pr create --base <target-branch> --title "..." --body-file tmp/pr-body-<branch>.md`
   - **GitLab**: `glab mr create --title "..." --description "$(cat tmp/pr-body-<branch>.md)"`
1. Clean up the temp file: `rm tmp/pr-body-<branch>.md`
