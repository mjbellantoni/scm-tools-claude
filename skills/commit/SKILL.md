---
name: commit
description: Stage and commit changes following git commit standards
---

# Commit Changes

Stage and commit changes with proper commit message formatting.

## Git Executable

If `bin/scm-tools-git` exists in the project root, use `bin/scm-tools-git` instead of `git` for the add and commit steps. All other git commands (status, diff, log, etc.) use plain `git`.

## Workflow

1. **Review changes:**
   ```bash
   git status
   git diff
   git diff --staged
   ```

2. **Stage explicitly:**
   ```bash
   bin/scm-tools-git add app/models/user.rb
   bin/scm-tools-git add spec/models/user_spec.rb
   ```
   Or interactively: `bin/scm-tools-git add -p`

3. **Verify staging:**
   ```bash
   git diff --staged
   ```

4. **Atomicity check:** Draft your commit message, then invoke the
   commit-splitter agent with the staged diff and proposed message.
   If the verdict is "split", unstage everything, stage only the
   first unit, and restart from step 2. Commit each unit separately.
   If "atomic", proceed to step 5.

5. **Commit with heredoc:**
   ```bash
   git commit -m "$(cat <<'EOF'
   Subject line here

   Optional body explaining why, wrapped at 72 chars.
   EOF
   )"
   ```

6. **Verify:**
   ```bash
   git log --oneline -1
   ```

---

## Git Tool Usage (Safety Rules)

### Staging & Unstaging

| Task     | Do                             | Don't                     |
| -------- | ------------------------------ | ------------------------- |
| Stage    | `bin/scm-tools-git add <path>`, `bin/scm-tools-git add -p` | `git add -A`, `git add .` |
| Unstage  | `git restore --staged <path>`  | `git reset <path>`        |

- Stage explicitly; no blanket adds
- Use `bin/scm-tools-git add -p` to split changes into logical commits

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
8. **ASCII only** — no Unicode dashes (—, –), quotes (" " ' '), arrows (→), or other non-ASCII characters. Use plain hyphens (-), straight quotes (' "), and ASCII arrows (->)

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
Add validation for "email"       # smart quotes, use straight quotes
Fix timeout — increase to 30s    # em dash, use hyphen
Update deps → patch CVE          # Unicode arrow, use ->
Address review feedback          # kitchen-sink, split into separate commits
Fix lint errors across files     # kitchen-sink, one commit per file/concern
Add validation; update tests     # semicolons joining unrelated changes
```

## Body Voice and Structure

The subject line is imperative and subject-elided ("Add X"). The body is
not. Write the body in **full declarative sentences with explicit
subjects** -- it should sound like a person explaining the change to a
colleague, not a changelog entry.

- **Restore subjects.** Every sentence needs a noun doing the verb.
- **One paragraph per facet.** A commit touching three facets of one
  concern gets three short paragraphs, not one dense block.
- **Wrap at 72 characters.** (Same rule as the subject line section.)
- **Conversational register is fine.** Parenthetical asides, hedges,
  and natural phrasing all belong in the body.

### Body anti-patterns

These smells mean the subject line's register has leaked into the body:

| Anti-pattern | Example |
| --- | --- |
| Subject-elided imperatives | "Mirrors X." "Gains Y." "Threads Z through W." |
| Joiners doing a word's work | `+`, `&`, `;` connecting clauses, or "plus" gluing thoughts |
| Wall of text | One paragraph covering three or more distinct facets |
| Clipped staccato cadence | "X does Y. Y does Z. Z does W." across consecutive sentences |

### Before and after

**Bad** -- imperative fragments, no subjects, single paragraph:
```
Mirrors PasswordsController's shape with param: :token and a
check_inbox action. Views commit to form+button semantics,
aria-labelledby landmarks, and content_for titles. Admin layout
gains <main> landmark and dynamic <title>.
```

**Good** -- declarative sentences, explicit subjects, one paragraph per facet:
```
The controller mirrors PasswordsController's shape - new, create,
edit, update - with param: :token and a check_inbox collection
action.

The views stick to form-and-button semantics rather than
Turbo-method links, use aria-labelledby landmarks, and set titles
via content_for. The admin layout picks up a <main> landmark and
a dynamic <title>.
```

**Bad** -- telegraphic "gains", semicolon-joined facets:
```
Extends sign_in_as_admin with a with_elevation: kwarg so existing
admin specs stay green, and threads with_elevation: true through
the impersonation request spec plus the users spec. Users spec
gains a redirect assertion; session_elevations spec gains the
return-to round-trip and cross-session PATCH rejection tests.
```

**Good** -- split into paragraphs, natural verbs:
```
To keep existing admin specs green, sign_in_as_admin takes a new
with_elevation: kwarg. The impersonation request spec and the
users spec both pass with_elevation: true.

The users spec also picks up a redirect assertion for the
unelevated case. The session_elevations spec now covers the
return-to round-trip end-to-end and verifies that a PATCH from
a different session is rejected.
```

Note: multi-paragraph bodies describe multiple facets of **one concern**.
They are not license to bundle unrelated concerns into a single commit.

## One Concern Per Commit

**Each commit must address exactly one concern.** If you're about to commit
changes that span multiple concerns, STOP and split them into separate commits.

### How to detect a kitchen-sink commit

Before committing, draft your commit message mentally. If any of these are
true, you need multiple commits:

- Your message has **bullet points** listing different changes
- Your message uses **semicolons** or **comma-separated lists** of distinct concerns
- Your message says "and" joining unrelated changes ("Fix lint errors and update API endpoint")
- You can't summarize it in a single imperative sentence under 50 characters
- The changes touch unrelated files for unrelated reasons

### Splitting by concern

- Separate refactoring from feature work
- Separate test additions from implementation
- Separate dependency updates from code changes
- Separate lint/style fixes from logic changes
- **Each independent fix is its own commit** -- even if they came from the same review round

### Review feedback commits

When implementing code review feedback, **each fix is its own commit** --
not one commit per review round. Stage and commit each fix before moving to
the next one. This makes review cleanup easier and keeps the history
reviewable.

```
# BAD: one commit for a review round
"Address review feedback: fix nil check, rename method, add test"

# GOOD: three separate commits
"Guard against nil user in checkout"
"Rename process_order to fulfill_order"
"Add test for expired coupon edge case"
```

## Output

After committing, report:
- What was committed (files, summary)
- The commit hash
- The commit message used
