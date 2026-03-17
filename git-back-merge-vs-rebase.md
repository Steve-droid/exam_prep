# Back-Merge vs Rebase

## Question
What is the difference between back-merge and rebase? What would you recommend?

## Ideal One-Liner
Back-merge brings main into your branch by creating a merge commit (safe, preserves history); rebase replays your commits on top of main creating new commit SHAs (linear history, but unsafe on shared branches).

---

## Full Answer

### The Setup — A Diverged Branch

Both techniques solve the same problem: your feature branch is behind main.

```
Initial state:

main:     A ── B ── C ── D        ← D was added while you worked
                    │
feature:            └── E ── F    ← your work
```

---

### Back-Merge (Merging main INTO your branch)

```bash
git checkout feature
git merge main
```

Git performs a **three-way merge**: it finds the common ancestor (C), combines D (from main) and E+F (from feature), and creates a new merge commit M.

```
After back-merge:

main:     A ── B ── C ── D
                    │         \
feature:            └── E ── F ── M    ← M has two parents: D and F
```

**Characteristics:**
- Creates a merge commit M.
- Original commits E and F **keep their SHA hashes**.
- History is non-linear but truthfully represents what happened.
- **Safe on shared branches** — history is only added to, never rewritten.

---

### Rebase (Replaying your commits on top of main)

```bash
git checkout feature
git rebase main
```

Git takes your commits (E, F) and **replays them one by one on top of D**, creating brand-new commits E' and F' with new SHAs.

```
Before rebase:

main:     A ── B ── C ── D
                    │
feature:            └── E ── F

After rebase:

main:     A ── B ── C ── D
                              \
feature:                       E' ── F'    ← new commits, new SHAs
```

The original E and F are abandoned. E' and F' contain the same *changes* but are technically different commits.

**Characteristics:**
- No merge commit — history is perfectly linear.
- Commits get **new SHAs** — this is history rewriting.
- **NOT safe on shared branches** — if anyone else has your old E and F commits, their history diverges from yours after rebase.

---

### Recommendation

| Scenario | Recommendation |
|---|---|
| Solo feature branch (only you use it) | **Rebase** — clean linear history, no noise |
| Branch shared with other developers | **Back-merge** — safe, preserves everyone's history |
| About to open a PR | **Rebase** first to get a clean, linear diff that's easy to review |
| After a PR is merged, keeping long-lived branch up to date | **Back-merge** — safer since others may have based work on it |

**My recommendation:** Use rebase by default for personal feature branches before opening a PR. Once a branch is shared or merged, never rebase it — use back-merge if you need to sync it.

> "Rebase locally, merge publicly."

---

### The Golden Rule of Rebase
**Never rebase a branch that others are working on.** Rebase rewrites commit SHAs. If teammate Alice has based her work on your old commit E, and you rebase to get E', Alice's branch now has a "ghost" parent that no longer officially exists. When she tries to push or merge, Git sees diverged histories and things get messy.
