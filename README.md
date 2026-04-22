# scm-tools

A [Claude Code plugin](https://docs.anthropic.com/en/docs/claude-code/plugins) providing git workflow skills for commits, pull requests, and branch management.

## Skills

### `/commit`

Stage and commit changes with proper formatting. Enforces explicit file staging (no `git add -A`), imperative-mood subject lines capped at 50 characters, and body text wrapped at 72 characters. Includes safety rules for staging, file operations, and history rewrites.

### `/pr`

Create a pull request for the current branch. Automatically checks for fixup commits (WIP, review fixes, lint fixes) and invokes `/cleanup-commits` before proceeding. The PR title follows commit message rules (important for squash merges), and the body includes a summary and test plan. Uses `gh pr create` under the hood.

### `/squash-merge`

Squash merge a PR via GitHub's API using `gh pr merge --squash`. Two-phase workflow: drafts a commit message following the project's commit standards, then waits for explicit approval before merging. Not for rebase merges.

### `/cleanup-commits`

Reorganize messy branch commits into clean, reviewable history before opening a PR or after review rounds. Uses a three-bucket model (extracted, feature, junk drawer) with an analyze-then-confirm loop. Includes pre-flight safety checks for file overlap before reordering and multi-pass rebase execution with diff verification.

### `/cleanup-worktree`

Post-merge branch cleanup. Fetches the latest main, detaches HEAD at that commit, and deletes the local working branch. Safe by default — uses `git branch -d` and asks before force-deleting unmerged branches.

### `/land-pr`

End-to-end PR landing workflow for worktrees. Merges the PR (rebase by default, squash when requested), deletes the remote branch, invokes `/cleanup-worktree` for local reset, and finishes the Trello card. Fail-fast — stops on any error.

## Agents

### `commit-splitter`

Enforces one-concern-per-commit atomicity. Invoked during `/commit` with the staged diff and proposed message. Returns `{"verdict": "atomic"}` or `{"verdict": "split", "units": [...]}`.

### `commit-message-reviewer`

Checks a proposed commit message for mechanical rule violations. Designed to run as a pre-commit hook. Runs 11 checks: subject length, trailing period, capitalization, imperative mood, non-ASCII characters, umbrella language, body line length, body non-ASCII, forbidden footers, subject-elided verbs, and semicolons joining facets. Returns `{"verdict": "pass"}` or `{"verdict": "fail", "issues": [...]}`.

## Installation

Add the plugin to your Claude Code settings (`~/.claude/settings.json` or project-level):

```json
{
  "plugins": [
    {
      "type": "github",
      "url": "https://github.com/mjb/scm-tools-claude"
    }
  ]
}
```

## Commit Message Rules

All skills that produce commit messages follow the same standards:

1. Subject line 50 characters or fewer
2. Capitalize the subject
3. No period at the end
4. Imperative mood ("Add", "Fix", "Update")
5. Body wrapped at 72 characters
6. Explain what and why, not how
7. No branding or footers
