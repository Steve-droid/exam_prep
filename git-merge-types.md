# Git Merge Types

## Question
Explain the different types of merges in Git: fast-forward, three-way merge, rebase, and back-merge.

## Ideal One-Liner
Git has four integration strategies: fast-forward (pointer moves, no commit), three-way merge (merge commit from two tips + ancestor), rebase (replays commits as new SHAs for linear history), and back-merge (merging the target branch into your branch to stay updated).

---

## Full Answer

---

### 1. Fast-Forward Merge

**When it occurs:** The branch being merged into has not moved since the feature branch was created — there is a direct linear path from the target to the tip of the feature branch.

```
Before:

main:     A ── B
                \
feature:         C ── D

(main is at B; B is a direct ancestor of D — no divergence)

After `git merge feature`:

main:     A ── B ── C ── D    ← main pointer just moved to D
```

- **No new merge commit is created.**
- History stays perfectly linear.
- Condition required: target branch must be a direct ancestor of the branch being merged.
- If you want to force a merge commit anyway: `git merge --no-ff feature`
- If you want it to fail rather than do a three-way merge: `git merge --ff-only feature`

---

### 2. Three-Way Merge

**When it occurs:** Both branches have new commits since they diverged — fast-forward is not possible.

Git uses **three snapshots** to compute the result:
1. The **common ancestor** (the point where they diverged).
2. The **tip of the current branch** (main).
3. The **tip of the incoming branch** (feature).

```
Before:

main:     A ── B ── C            ← C added after feature branched
                    │
feature:  A ── B ── D ── E

After `git merge feature`:

main:     A ── B ── C ─────── M  ← M is the merge commit
                    │         │
feature:            └── D ── E
                             ↑
                         (M's second parent)
```

- Creates a **merge commit M** with two parents (C and E).
- History is non-linear — you can see where branches diverged and rejoined.
- This is Git's default behavior when fast-forward isn't possible.

---

### 3. Rebase

**When it occurs:** You explicitly choose `git rebase` instead of `git merge`.

Rebase takes commits from your branch and **replays them one by one on top of the target branch**, creating brand-new commits (D', E') with new SHAs.

```
Before:

main:     A ── B ── C
                    │
feature:            └── D ── E

After `git rebase main` (on feature branch):

main:     A ── B ── C
                         \
feature:                  D' ── E'   ← new commits, new SHAs

(original D and E are abandoned)
```

- **No merge commit.** History is linear.
- Commits get **new SHA hashes** — this is history rewriting.
- Safe only for **private, unshared branches**.
- Makes PRs cleaner and easier to review — linear story of changes.

Why D' and not D? Because a commit's SHA includes its parent's SHA. When the parent changes (now C instead of B), the SHA of the replayed commit changes — it's technically a new commit object.

---

### 4. Back-Merge

**When it occurs:** You want to bring updates from the main (or target) branch INTO your feature branch, while keeping your branch active.

```
Before:

main:     A ── B ── C ── D    ← D was added to main while you worked
                    │
feature:            └── E ── F

After `git merge main` (on feature branch):

main:     A ── B ── C ── D
                    │         \
feature:            └── E ── F ── M    ← M merges D into feature
```

- Creates a merge commit M on the **feature branch** (not on main).
- Feature branch is now up to date with main.
- **Safe on shared branches** — original commits are preserved.
- Use when: a critical fix landed on main that your feature depends on, or when preparing for a final merge and you want to resolve conflicts on your branch rather than on main.

---

### Summary Table

| Type | Merge Commit? | Rewrites History? | Safe on Shared Branches? |
|---|---|---|---|
| Fast-forward | No | No | Yes |
| Three-way merge | Yes | No | Yes |
| Rebase | No | Yes (new SHAs) | No — solo branches only |
| Back-merge | Yes (on feature) | No | Yes |
