---
name: commit
description: Stage and commit changes following git commit standards
---

# Commit Changes

Stage and commit changes with proper commit message formatting.

## Workflow

1. **Review changes:**
   ```bash
   git status
   git diff
   git diff --staged
   ```

2. **Stage explicitly:**
   ```bash
   git add app/models/user.rb
   git add spec/models/user_spec.rb
   ```
   Or interactively: `git add -p`

3. **Verify staging:**
   ```bash
   git diff --staged
   ```

4. **Commit with heredoc:**
   ```bash
   git commit -m "$(cat <<'EOF'
   Subject line here

   Optional body explaining why, wrapped at 72 chars.
   EOF
   )"
   ```

5. **Verify:**
   ```bash
   git log --oneline -1
   ```

---

## Git Tool Usage (Safety Rules)

### Staging & Unstaging

| Task     | Do                             | Don't                     |
| -------- | ------------------------------ | ------------------------- |
| Stage    | `git add <path>`, `git add -p` | `git add -A`, `git add .` |
| Unstage  | `git restore --staged <path>`  | `git reset <path>`        |

- Stage explicitly; no blanket adds
- Use `git add -p` to split changes into logical commits

### File Operations

| Task           | Do                    | Don't            |
| -------------- | --------------------- | ---------------- |
| Delete tracked | `git rm [-r] <path>`  | `rm <path>`      |
| Rename/move    | `git mv <old> <new>`  | `mv <old> <new>` |

### History Safety

- Use `git revert <sha>` to undo merged history on shared branches
- If you must force push, use `--force-with-lease` (never plain `--force`)
- Never run `git reset --hard` on shared work without confirmation

---

## Commit Message Rules

1. **Subject line ≤50 characters**
2. **Capitalize the subject**
3. **No period at the end**
4. **Imperative mood** — "Add", "Fix", "Update" (not "Added", "Fixed")
5. **Wrap body at 72 characters** — insert line breaks manually
6. **Explain what and why, not how**
7. **NEVER add branding or footers** — no Co-Authored-By, no signatures

### Good Examples
```
Add validation for email format
Fix login timeout on slow connections
Update dependencies to patch CVE-2024-1234
```

### Bad Examples
```
Fixed the bug                    # past tense, vague
Updates to the login page.       # third person, period, vague
WIP                              # meaningless
```

## When to Split Commits

If changes span multiple concerns, make multiple commits:
- Separate refactoring from feature work
- Separate test additions from implementation
- Separate dependency updates from code changes

## Output

After committing, report:
- What was committed (files, summary)
- The commit hash
- The commit message used
