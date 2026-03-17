# Handling a Bad Commit on a Shared Branch

## Question
How do you handle a bad commit that has been pushed to a shared branch?

## Ideal One-Liner
On a shared branch, always use `git revert` — it creates a new commit that undoes the bad change without rewriting history, keeping everyone's local copies valid.

---

## Full Answer

### Pre-Push vs Post-Push — The Key Distinction

The right approach depends entirely on whether the commit has been pushed to a shared branch.

---

### Scenario 1: Bad Commit NOT Yet Pushed (safe to rewrite)

The commit only exists locally. No one else has it. You can freely rewrite history.

```bash
# Option A: remove the last commit, keep changes staged
git reset --soft HEAD~1

# Option B: remove the last commit, keep changes unstaged
git reset --mixed HEAD~1   # this is the default

# Option C: remove the last commit AND discard all changes
git reset --hard HEAD~1

# Option D: fix the commit (change message or add forgotten file)
git add forgotten_file.py
git commit --amend -m "Correct commit message"

# Then push normally
git push origin feature/my-branch
```

---

### Scenario 2: Bad Commit PUSHED to a Shared Branch (handle carefully)

#### The Safe Way: git revert

```bash
git revert <bad-commit-sha>
# or revert the last commit:
git revert HEAD
```

`git revert` creates a **new commit** that applies the inverse of the bad commit. The bad commit stays in history, but its effects are undone by the new revert commit.

```
Before:   A ── B ── C (bad) ── D
After:    A ── B ── C (bad) ── D ── R   ← R reverts C's changes
```

Why this is safe:
- History is never rewritten — everyone's local copies remain valid.
- The bad commit is still visible in history (audit trail).
- No force-push needed.
- Team members can `git pull` normally.

---

#### The Dangerous Way: Force Push (use with caution)

```bash
git reset --hard <commit-before-bad>
git push --force origin main
```

This **rewrites shared history**. The bad commit is gone, but:
- Anyone who pulled after the bad commit now has a diverged history.
- They will see errors on their next `git pull` and need to manually reset.
- If they push without resetting, the bad commit comes back.
- Can cause data loss if not coordinated properly.

**When force push is acceptable:**
- The branch is technically shared, but in practice only you are working on it (e.g., your PR branch).
- The team is small and you can coordinate: "I'm force-pushing feature/x — please re-pull."
- You are the only developer and it's a fresh branch.

**Never force-push to `main` or `develop`** in a team environment without explicit coordination.

---

### Decision Tree

```
Is the commit pushed to a shared branch?
├── No → git reset (soft/mixed/hard) or git commit --amend
└── Yes
    ├── Is anyone else actively using this branch?
    │   ├── Yes → git revert (safe, no force push)
    │   └── No (just you) → git reset + git push --force (with caution)
    └── Is it main/develop? → git revert ALWAYS
```

---

### Commands Quick Reference

| Situation | Command |
|---|---|
| Undo last unpushed commit, keep changes | `git reset --soft HEAD~1` |
| Undo last unpushed commit, discard changes | `git reset --hard HEAD~1` |
| Fix last commit message | `git commit --amend` |
| Undo a pushed commit safely | `git revert <sha>` |
| Nuke a pushed commit (dangerous) | `git reset --hard <sha>` + `git push --force` |
