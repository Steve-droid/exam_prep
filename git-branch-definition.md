# Git Branch Definition

## Question
How would you define a branch?

## Ideal One-Liner
A branch is a lightweight pointer — just a file containing a commit SHA — that moves forward automatically as new commits are made on it.

---

## Full Answer

### What a Branch Actually Is
A branch is **not a copy of files**. It is a tiny text file stored in `.git/refs/heads/` containing the SHA-1 hash of the commit it points to.

```bash
cat .git/refs/heads/main
# a1b2c3d4e5f6...   ← just a 40-character SHA
```

That's it. Creating a branch is nearly instantaneous and costs almost no disk space, regardless of repo size.

---

### How It Works — Commit History

```
A ── B ── C          ← main (points to C)
          │
          └── D ── E ← feature (points to E)
```

- `main` is a pointer to commit C.
- `feature` is a pointer to commit E.
- Commits D and E exist only on the feature branch — main doesn't know about them yet.
- When you merge or rebase, Git integrates these commit chains.

---

### What HEAD Is
`HEAD` is a special pointer that points to **the branch you currently have checked out** (or directly to a commit in "detached HEAD" state).

```
HEAD → main → C
```

When you make a new commit:
1. Git creates the commit object (pointing to its parent).
2. The **current branch pointer moves forward** to the new commit.
3. HEAD still points to the branch, so HEAD moves too.

```
Before:  HEAD → main → C
After:   HEAD → main → C → D   (new commit D)
```

---

### Detached HEAD State
If you checkout a specific commit directly (not a branch):
```bash
git checkout a1b2c3d
```
HEAD points to the commit SHA directly — not to a branch. Any new commits you make won't be tracked by any branch and can be lost when you switch away.

```
HEAD → a1b2c3d   (no branch)
```

To save work done in detached HEAD: `git checkout -b new-branch-name`

---

### Creating a Branch vs Copying Files
| Operation | What actually happens |
|---|---|
| `git checkout -b feature` | Creates `.git/refs/heads/feature` with current commit SHA. 0 files copied. |
| Copying a folder | Duplicates all files on disk. Slow and expensive. |

Branches are cheap because they are just pointers — the actual file snapshots live in the commit objects, which are shared.

---

### Branch Lifecycle Summary
1. `git checkout -b feature` — create branch, both `main` and `feature` point to same commit.
2. Make commits on `feature` — `feature` pointer moves forward, `main` stays put.
3. Merge/rebase — integrates `feature` into `main`.
4. `git branch -d feature` — deletes the pointer file. Commits remain in history until garbage collected.
