---
name: cleanup-commits
description: Use when reorganizing messy branch commits before PR or after review rounds - analyzes commits into buckets, proposes regrouping, and executes safe rebase with pre-flight overlap checks
---

# Cleanup Commits

Reorganize commits on a feature branch into clean, reviewable history using a three-bucket model with an analyze-then-confirm loop.

## Process

### 1. Detect merge base

```bash
git merge-base HEAD main
```

Store as `$MERGE_BASE`. If the branch has been rebased or has a non-standard base, confirm with user.

### 2. Analyze current commits

```bash
git log --oneline $MERGE_BASE..HEAD
```

For each commit, read the diff (`git show --unified=0 <sha>`) to understand what it touches. Track which files and which line ranges (from `@@ -a,b +c,d @@` hunk headers) each commit modifies — you will need this for the pre-flight check.

**Flag kitchen-sink commits for splitting.** Apply the heuristics from the "How to detect a kitchen-sink commit" section of `/commit`. Mark flagged commits in the proposal table with a note like "kitchen-sink -- propose split into N commits" and suggest individual commit messages for each concern.

**If every commit has a single concern, a valid message, and the ordering is already coherent, report "History is already clean -- no changes needed" and stop.** Do not manufacture improvements to justify running the skill. Churning on already-clean commits punishes good commit hygiene.

### 3. Propose regrouping

Present a table:

| Current commit(s) | Target bucket | Proposed message | Notes |
|---|---|---|---|

**Buckets:**
- **Extracted** — changes to shared/common code outside the PR's core scope. Clean, self-contained. Candidates for cherry-picking into own PR later.
- **Feature** — the actual work the branch exists for. Multiple commits fine if they represent meaningful steps. Review fixes that are substantive (guard clauses, renames, added tests) belong here if they're already well-split into one-concern-per-commit.
- **Junk drawer** — truly trivial fixups only: typo corrections, whitespace, formatting. Squash into one or a few commits. Do NOT lump substantive review fixes here just because they came from a review round.

Call out ambiguities explicitly: "This migration supports the feature but could also stand alone — where do you want it?"

Follow commit message rules from `/commit`.

### 4. STOP — wait for confirmation

**Do not touch git history until the user approves or adjusts the plan.**

Present the table and ask: "Does this grouping look right, or would you like to adjust anything?"

Do not proceed. Do not run any git commands that modify history. Wait.

### 5. Pre-flight safety check

Before executing any rebase, analyze the proposed operations for risk:

1. For each proposed reorder, check if commits being moved past each other touch **overlapping line ranges within the same files**. Use the hunk headers (`@@ -a,b +c,d @@`) collected in step 2 to compare ranges. Expand each range by 3 lines (git's default context width) before comparing — adjacent hunks can conflict even without strict overlap.
2. If overlapping ranges exist, report the specific overlapping regions and offer alternatives:
   - Leave in place, just improve the commit message
   - Squash without reordering
   - Proceed with reorder (user accepts conflict risk)
3. Rate risk for each proposed operation:

| Operation | Risk |
|---|---|
| Squash adjacent commits | Safe |
| Improve commit message only | Safe |
| Reorder commits touching different files | Low risk |
| Reorder commits touching same files, non-overlapping line ranges | Moderate risk — note for user |
| Reorder commits touching same files, overlapping line ranges | **High risk — warn user** |
| Split commits | **Highest risk — only if user explicitly asks** |

Present the risk assessment. **Wait for user to acknowledge before starting.**

### 6. Execute the rebase

**Before anything else:**
```bash
git branch backup/<branch-name>
```

**Execution strategy — prefer multiple safe passes over one risky pass:**

1. **Pass 1: Squash** (safe). Squash adjacent commits per the approved plan. After completing:
   ```bash
   git diff $MERGE_BASE..HEAD
   ```
   Compare against pre-rebase diff. If different, something went wrong — abort.

2. **Pass 2: Reorder** (risky). Only if approved in pre-flight. Reorder commits per plan. After completing, verify diff again.

3. **Pass 3: Message cleanup** (safe). Amend commit messages as needed.

**After each pass**, verify:
```bash
git diff $MERGE_BASE..backup/<branch-name> --stat
git diff $MERGE_BASE..HEAD --stat
```
Both should show identical file-level changes.

**If a conflict occurs during rebase:** STOP immediately.
- Show the conflict to the user
- Do NOT auto-resolve, even if the resolution seems obvious
- Walk the user through resolution options
- If they want to abort: `git rebase --abort`

### 7. Show final result

```bash
git log --oneline $MERGE_BASE..HEAD
```

Confirm with the user. Remind them:
- Backup branch `backup/<branch-name>` exists and can be deleted once satisfied
- If anything looks wrong: `git reset --hard backup/<branch-name>`
- If they need to force-push, they should confirm explicitly

## Red Flags — STOP If You Catch Yourself Doing These

- **Rebasing before user approves the plan** — "They said ASAP" or "the plan is obvious" is not approval
- **Reordering without checking line-range overlap** — "It should be fine" is not a safety check
- **Auto-resolving conflicts** — "The resolution is obvious" or "it's just a mechanical merge" is not your call
- **Force-pushing without explicit confirmation** — never
- **Skipping the backup branch** — never
- **Skipping diff verification after a rebase pass** — always verify
- **Proposing changes when history is already clean** — "there's always something to improve" is not a reason to churn

| Rationalization | Reality |
|---|---|
| "User said ASAP, no time for confirmation" | ASAP means fast analysis, not skipping safety |
| "The reorder is safe, the line ranges barely overlap" | Barely overlap = overlap. Warn the user. |
| "Resolution is obvious, no need to stop" | Obvious to you ≠ what the user intended |
| "Waiting serves no purpose" | Your judgment on their code is not authoritative |
| "It's just a mechanical merge" | Mechanical merges can silently drop logic |
| "Low risk, easily fixable with abort" | Abort mid-rebase can leave messy state. Prevent, don't recover. |
| "This kitchen-sink commit is fine as-is" | If the message needs bullets or semicolons, it's multiple commits. Propose a split. |
| "There's always something to improve" | If commits are clean, single-concern, and well-ordered, stop. Manufacturing changes undermines good commit hygiene. |

## Quick Reference

| Phase | Action | Gate |
|---|---|---|
| 1. Detect | `git merge-base HEAD main` | — |
| 2. Analyze | `git log`, `git show` per commit | If already clean, **stop** |
| 3. Propose | Present bucket table | — |
| 4. Confirm | **STOP** — wait for user | **User approves** |
| 5. Pre-flight | Check line-range overlaps, rate risk | **User acknowledges risk** |
| 6. Execute | Backup → squash → reorder → messages | Verify diff each pass |
| 7. Finalize | Show result, note backup exists | **User confirms** |
