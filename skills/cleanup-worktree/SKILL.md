---
name: cleanup-worktree
description: Use when work is complete and PR is merged to clean up the worktree - fetches main, detaches HEAD, and deletes the working branch
---

# Cleanup Worktree

## Overview

After a PR is merged, clean up the worktree by fetching the latest main,
detaching HEAD at that commit, and deleting the local working branch.

## The Process

### Phase 1: Identify Current Branch

```bash
git branch --show-current
```

Store the branch name. If already detached or on main, warn and stop.

### Phase 2: Fetch Latest Main

```bash
git fetch origin main
```

### Phase 3: Detach HEAD at Main

```bash
git checkout --detach origin/main
```

### Phase 4: Delete Working Branch

```bash
git branch -d <branch-name>
```

If the branch has unmerged changes, git will refuse with `-d`. Ask user
if they want to force delete with `-D`.

### Phase 5: Confirm

Report: "Worktree cleaned up. HEAD detached at origin/main, branch `<name>` deleted."

## Red Flags

If you catch yourself doing these, STOP:

- **Deleting main or master** - Never delete these branches
- **Force deleting without asking** - Always try `-d` first, ask before `-D`
- **Running on non-worktree directories** - This is for worktree cleanup only

## Quick Reference

| Phase | Command | Purpose |
|-------|---------|---------|
| 1. Identify | `git branch --show-current` | Get branch to delete |
| 2. Fetch | `git fetch origin main` | Get latest main |
| 3. Detach | `git checkout --detach origin/main` | Detach at main |
| 4. Delete | `git branch -d <branch>` | Remove working branch |
| 5. Confirm | Report status | Done |
