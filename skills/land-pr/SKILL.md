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

### 2. Merge the PR

**Rebase merge (default):**

```bash
gh pr merge --rebase
```

**Squash merge** (when user explicitly requests squash):

Invoke `scm-tools:squash-merge`. This handles commit message drafting and
user approval before merging.

### 3. Delete remote branch

```bash
git push origin --delete <branch-name>
```

GitHub may auto-delete depending on repo settings. If the command fails
because the branch is already gone, that is expected — continue to step 4.

### 4. Clean up locally

Invoke `scm-tools:cleanup-worktree`.

Fetches main, detaches HEAD at origin/main, deletes the local branch.

### 5. Finish Trello card

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
| 2a | Rebase merge (default) | `gh pr merge --rebase` |
| 2b | Squash merge (if requested) | `scm-tools:squash-merge` |
| 3 | Delete remote branch | `git push origin --delete <branch>` |
| 4 | Local cleanup | `scm-tools:cleanup-worktree` |
| 5 | Finish Trello card | `trello-cli:finish-card` |
