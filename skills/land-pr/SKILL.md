---
name: land-pr
description: Use when a PR is approved and ready to merge, and the worktree needs to be cleaned up for reuse afterward
---

# Land PR

Orchestrates the full workflow for merging an approved PR and resetting the
worktree for reuse. Fail-fast — if any step fails, report and stop.

## Steps

### 1. Capture branch name

```bash
git branch --show-current
```

Store this BEFORE any other step. You need it for remote branch deletion
after the merge, and checkout operations in later steps will lose it.

### 2. Commit hygiene check

For rebase merges (where individual commits are preserved), scan for fixup
commits:

```bash
git log --oneline $(git merge-base HEAD main)..HEAD
```

If any commit messages match these patterns (case-insensitive), invoke
`scm-tools:cleanup-commits` before merging:
- `fixup!` or `squash!`
- `WIP`
- `address review` or `review feedback`
- `lint`, `rubocop`, `typo` as standalone fixes

Skip this check for squash merges (history is collapsed anyway).

### 3. Merge the PR

**Rebase merge (default):**

```bash
gh pr merge --rebase
```

**Squash merge** (when user explicitly requests squash):

Invoke `scm-tools:squash-merge`. This handles commit message drafting and
user approval before merging.

### 4. Delete remote branch

```bash
git push origin --delete <branch-name>
```

GitHub may auto-delete depending on repo settings. If the command fails
because the branch is already gone, that is expected — continue to step 5.

### 5. Clean up locally

Invoke `scm-tools:cleanup-worktree`.

Fetches main, detaches HEAD at origin/main, deletes the local branch.

### 6. Finish Trello card

Invoke `trello-cli:finish-card`.

Always run this step — it is part of the workflow even when the user does
not mention Trello explicitly.

## Critical Rules

- **Capture branch name first** — before any checkout, detach, or skill
  invocation that might change HEAD
- **Never delete or recreate worktree directories** — worktrees are
  recycled and their directory names are keys in other systems
- **Stop on failure** — if any step fails, report the failure and stop
  immediately. Do not continue past a failed step.

## Quick Reference

| Step | Action | Tool |
|------|--------|------|
| 1 | Capture branch name | `git branch --show-current` |
| 2 | Commit hygiene check | `git log`, then `scm-tools:cleanup-commits` if needed |
| 3a | Rebase merge (default) | `gh pr merge --rebase` |
| 3b | Squash merge (if requested) | `scm-tools:squash-merge` |
| 4 | Delete remote branch | `git push origin --delete <branch>` |
| 5 | Local cleanup | `scm-tools:cleanup-worktree` |
| 6 | Finish Trello card | `trello-cli:finish-card` |
