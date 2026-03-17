# git reset vs git revert vs git checkout

## Question
What is the difference between git reset, git revert, and git checkout? When should each be used?

## Ideal One-Liner
`git reset` moves the branch pointer backward (rewrites history, local only); `git revert` creates a new undo-commit (safe for shared branches); `git checkout` switches branches or restores files (doesn't affect history at all).

---

## Full Answer

### Overview

| Command | Affects commit history? | Rewrites history? | Safe on shared branches? |
|---|---|---|---|
| `git reset` | Yes — moves branch pointer | Yes | No |
| `git revert` | Yes — adds a new commit | No | Yes |
| `git checkout` | No | No | Yes |

---

### git reset

Moves the **HEAD and branch pointer** to a specified previous commit. Everything after that commit is "undone" — but what happens to the changes depends on the flag.

```
Before:   A ── B ── C ── D    ← HEAD/main at D
After reset to B:
          A ── B              ← HEAD/main now at B
                (C and D are no longer referenced by any branch)
```

**Three modes:**

```bash
git reset --soft HEAD~1    # Move pointer back 1; changes from D go to STAGING AREA
git reset --mixed HEAD~1   # Move pointer back 1; changes from D go to WORKING DIR (default)
git reset --hard HEAD~1    # Move pointer back 1; changes from D are DISCARDED ENTIRELY
```

| Mode | Commit history | Staging area | Working directory |
|---|---|---|---|
| `--soft` | Moved back | Staged (changes preserved) | Unchanged |
| `--mixed` | Moved back | Cleared | Changes preserved (unstaged) |
| `--hard` | Moved back | Cleared | Cleared (changes GONE) |

**When to use:** Only for local, unpushed commits. Great for cleaning up messy commit history before pushing. `--hard` should be used with care — changes are unrecoverable (unless they were previously committed).

---

### git revert

Creates a **new commit** that is the exact inverse of a specified commit. The bad commit remains in history, but its effects are cancelled out.

```bash
git revert abc1234      # revert specific commit by SHA
git revert HEAD         # revert the most recent commit
git revert HEAD~2       # revert the commit two steps back
```

```
Before:   A ── B ── C (bad) ── D
After git revert C:
          A ── B ── C (bad) ── D ── R    ← R is the revert commit
```

**When to use:**
- When the commit has already been pushed to a shared branch.
- When you want an audit trail showing the mistake and its correction.
- Always prefer revert over reset on `main`, `develop`, or any shared branch.

---

### git checkout

Originally did multiple unrelated things. In modern Git (2.23+), its two main uses are split into dedicated commands:

**1. Switching branches:**
```bash
git checkout feature/my-branch
# Modern equivalent:
git switch feature/my-branch
```

**2. Restoring a specific file to its last committed state (discarding working dir changes):**
```bash
git checkout -- path/to/file.py
# Modern equivalent:
git restore path/to/file.py
```

**3. Detached HEAD — checking out a specific commit:**
```bash
git checkout abc1234       # HEAD now points directly to a commit, not a branch
```

**When to use:**
- Switch between branches during normal development.
- Discard changes to a specific file you don't want to keep.
- Inspect an old commit temporarily (detached HEAD).

`git checkout` **does not change the commit history** — it just moves HEAD or updates your working directory.

---

### Practical Scenarios

| Scenario | Command |
|---|---|
| Undo last commit, keep changes for editing | `git reset --soft HEAD~1` |
| Completely undo last local commit | `git reset --hard HEAD~1` |
| Undo a pushed commit safely | `git revert <sha>` |
| Discard changes to one file | `git restore path/to/file` |
| Switch to another branch | `git switch branch-name` |
| See what an old commit looked like | `git checkout <sha>` (read-only inspection) |
