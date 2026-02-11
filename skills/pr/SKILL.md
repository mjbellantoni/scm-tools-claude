---
name: pr
description: Create a pull request with proper formatting
---

# Create Pull Request

Create a pull request for the current branch, following commit message rules for the title (important for squash-merge).

## PR Title Rules

The PR title becomes the commit message when squash-merging. Follow these rules:

1. **Limit to 50 characters** (hard limit for commit subjects)
2. **Capitalize the first word**
3. **No period at the end**
4. **Use imperative mood** ("Add", "Fix", "Update" — not "Added", "Fixed")
5. **Explain *what*, not *how***

### Good Examples
- `Add validation for email format`
- `Fix login timeout on slow connections`
- `Update dependencies to patch security issue`

### Bad Examples
- `Fixed the bug` (past tense, vague)
- `Updates to the login page.` (third person, period, vague)
- `WIP: trying to fix auth` (WIP, lowercase, vague)

## PR Body

Include:
- **Summary**: 1-3 bullet points of what changed and why
- **Test plan**: How to verify the changes work

## Process

0. If `brivlo:send_event` is available, use `brivlo:send_event scm-tools:pr` as your first step.

1. Check current branch status and commits since diverging from base
2. Determine base branch (use argument if provided, otherwise `main`)
3. Push branch to remote if needed
4. Create PR with properly formatted title and body using `gh pr create`
5. Return the PR URL
