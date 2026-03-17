# Git Fast-Forward: --ff-only and --no-ff

## Question
What does --ff-only mean? --no-ff? When should each be used?

## Ideal One-Liner
`--ff-only` merges by simply moving the branch pointer forward (no merge commit) and fails if that's not possible; `--no-ff` always creates a merge commit, preserving the fact that a branch existed even when fast-forward would have been possible.

---

## Full Answer

### What Is a Fast-Forward Merge?
A fast-forward is possible when the target branch has **no new commits** since the feature branch was created — the branch history is linear, and the target is a direct ancestor of the feature tip.

```
Before merge:

main:    A ── B
                \
feature:         C ── D

(main is at B, feature is at D; B is a direct ancestor of D)
```

Git doesn't need to create a merge commit — it just moves `main` forward to D:

```
After fast-forward merge:

main:    A ── B ── C ── D     ← main pointer moved to D
```

No merge commit. History stays linear.

---

### --ff-only

```bash
git merge --ff-only feature
```

- **Performs the merge only if fast-forward is possible.**
- If `main` has moved forward since `feature` branched off, the command **fails with an error** rather than creating a merge commit.
- Use case: enforcing a clean linear history. If you want to guarantee no merge commits ever appear on main, use `--ff-only` and require developers to rebase their branches before merging.

```
If main has diverged:

main:    A ── B ── E          ← E was added to main
                \
feature:         C ── D

git merge --ff-only feature
→ ERROR: Not possible to fast-forward, aborting.
```

---

### --no-ff (No Fast-Forward)

```bash
git merge --no-ff feature
```

- **Always creates a merge commit**, even when fast-forward would have been possible.
- Preserves the fact that a feature branch existed in the history.
- The branch topology remains visible in `git log --graph`.

```
Before:

main:    A ── B
                \
feature:         C ── D

After --no-ff:

main:    A ── B ──────── M    ← M is the merge commit
                \       /
feature:         C ── D
```

Merge commit M has two parents: B and D.

---

### When to Use Each

| Flag | Use when |
|---|---|
| `--ff-only` | You want a strictly linear history on main. Pairs with a rebase-before-merge workflow. Good for teams that value readable `git log`. |
| `--no-ff` | You want to preserve branch context. Useful in Git Flow where you want to see "this chunk of commits was a feature branch." Default behavior of GitHub PRs merge button. |
| (default, no flag) | Fast-forward if possible, three-way merge otherwise. Fine for casual use, inconsistent history style. |

---

### Summary Table

| Scenario | --ff-only | --no-ff |
|---|---|---|
| Branch is ahead of main, no divergence | Succeeds, no merge commit | Succeeds, creates merge commit |
| Main has diverged from branch | **Fails** | Creates merge commit with three-way merge |
| History style | Linear | Shows branch topology |
| When to use | Enforce rebase discipline | Preserve feature branch context |
