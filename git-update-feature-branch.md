# Updating a Feature Branch When Main Has Changed

## Question
You're working on a feature branch and your colleague tells you main has been updated with important changes. What do you do?

## Ideal One-Liner
If your branch is private, rebase onto main for a clean linear history; if your branch is shared with others, back-merge main into your branch to safely incorporate the updates.

---

## Full Answer

### The Situation

```
Initial state:

main:     A ── B ── C ── D    ← D is the important new change
                    │
feature:            └── E ── F   ← your work
```

You need D's changes in your feature branch. Two options:

---

### Option 1: Rebase (preferred for private branches)

```bash
git checkout feature/my-branch
git fetch origin
git rebase origin/main
```

Git replays your commits (E, F) on top of D:

```
After rebase:

main:     A ── B ── C ── D
                              \
feature:                       E' ── F'   ← new commits, new SHAs
```

**Result:** Linear history. E and F become E' and F' with new SHAs.

**Pros:**
- Clean, linear history — your feature sits neatly on top of main.
- No merge commit noise.
- PR diff is clean (only your changes, not a merge commit).

**Cons:**
- Rewrites commit history (new SHAs).
- **NOT safe if others have based work on your branch** — their history will diverge.
- If you've already pushed, you'll need `git push --force-with-lease` afterward.

```bash
# After rebase, force-push your branch (only if it's yours alone)
git push --force-with-lease origin feature/my-branch
```

Use `--force-with-lease` instead of `--force` — it fails if someone else has pushed to the branch since your last fetch, protecting against accidental overwrites.

---

### Option 2: Back-Merge (safe for shared branches)

```bash
git checkout feature/my-branch
git fetch origin
git merge origin/main
```

Git creates a merge commit M that incorporates D:

```
After back-merge:

main:     A ── B ── C ── D
                    │         \
feature:            └── E ── F ── M   ← M merges main into feature
```

**Result:** Non-linear history with a merge commit, but all original SHAs preserved.

**Pros:**
- Safe — no history rewriting.
- Works even if teammates are also working on this branch.
- No force-push needed.

**Cons:**
- Creates a merge commit, making history less linear.
- If done repeatedly, the branch history becomes cluttered with merge commits.

---

### The Decision

```
Is anyone else actively using this feature branch?
├── No (just me)
│   └── Is it already pushed?
│       ├── No → rebase freely
│       └── Yes → rebase + push --force-with-lease (communicate if needed)
└── Yes (shared branch)
    └── Back-merge (git merge origin/main) — always safe
```

---

### Commands Summary

```bash
# Rebase approach (private branch)
git fetch origin
git rebase origin/main
# resolve any conflicts during rebase, then:
git rebase --continue
git push --force-with-lease origin feature/my-branch

# Back-merge approach (shared branch)
git fetch origin
git merge origin/main
# resolve any conflicts, then:
git add .
git commit
git push origin feature/my-branch
```
