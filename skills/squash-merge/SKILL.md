---
name: squash-merge
description: Use when user explicitly wants to SQUASH merge a PR - drafts a commit message following commit rules, then squash merges via GitHub. Not for rebase merges.
---

# Squash Merge PR

## Overview

Squash merge a PR using GitHub (not local git), with a properly formatted
commit message that follows the project's commit standards.

**Use this skill only when the user explicitly requests a squash merge.**
Some PRs are rebased instead - this skill is not for those.

**This is a two-phase skill.** Phase 1 drafts the message and stops.
Phase 2 executes the merge after user approval.

## Phase 1: Draft Commit Message

### Step 1: Get PR Details

```bash
gh pr view --json title,body,commits,files
```

### Step 2: Review the Changes

Understand what the PR accomplishes by reviewing:
- PR title and description
- List of commits
- Files changed

### Step 3: Draft Commit Message

Follow the commit message rules from `/commit`:

1. **Subject line ≤50 characters**
2. **Capitalize the subject**
3. **No period at the end**
4. **Imperative mood** — "Add", "Fix", "Update" (not "Added", "Fixed")
5. **Wrap body at 72 characters** (if body needed)
6. **Explain what and why, not how**
7. **NEVER add branding or footers**

### Step 4: Present and STOP

Show the drafted message to the user:

```
Proposed commit message:
───────────────────────
<subject line>

<optional body>
───────────────────────
```

**STOP HERE. Take no further action until the user approves.**

## Phase 2: Execute Merge

Only proceed when the user explicitly approves the message.

### Step 1: Squash Merge via GitHub

```bash
gh pr merge --squash --subject "<subject>" --body "<body>"
```

- Use `--subject` for the first line
- Use `--body` for the rest (if any)
- Add `--delete-branch` if user wants remote branch deleted

**IMPORTANT:** Do NOT perform local git operations. This uses GitHub's
merge, not local squash.

### Step 2: Confirm

Report:
- PR merged successfully
- Final commit message used
- Whether branch was deleted

## Red Flags

If you catch yourself doing these, STOP:

- **Merging without approval** - Always show message first and wait
- **Using local git merge/squash** - Use `gh pr merge` only
- **Adding Co-Authored-By or footers** - Never add branding
- **Subject over 50 chars** - Shorten it
- **Past tense verbs** - Use imperative mood
- **Using this for rebase merges** - This skill is for squash only

## Quick Reference

| Phase | Action | Output |
|-------|--------|--------|
| 1. Get PR | `gh pr view --json ...` | PR details |
| 1. Draft | Follow commit rules | Message draft |
| 1. Present | Show message | **STOP** |
| 2. Merge | `gh pr merge --squash` | PR merged |
| 2. Confirm | Report result | Done |
