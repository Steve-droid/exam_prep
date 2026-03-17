# Git Conflicts

## Question
What is a conflict? When does it happen? How should it be handled?

## Ideal One-Liner
A merge conflict occurs when two branches have made different changes to the same lines of the same file, and Git cannot automatically decide which version to keep — it requires human resolution.

---

## Full Answer

### Merge vs Merge Conflict — They Are Not the Same

**A merge** always happens when combining two branches. Most merges succeed automatically:
- Changes are in different files → Git merges them without issue.
- Changes are in the same file but different lines → Git merges them without issue.

**A merge conflict** only occurs when:
- Two branches changed **the same lines** in the same file in **different ways**.
- Git cannot decide which version is "correct" — that's a human judgment.

```
Branch A changed line 10 to: "color: red"
Branch B changed line 10 to: "color: blue"
→ CONFLICT: Git doesn't know which one you want.
```

---

### What a Conflict Looks Like in the File

```
<<<<<<< HEAD
color: red
=======
color: blue
>>>>>>> feature/theme
```

- Everything between `<<<<<<< HEAD` and `=======` is **your current branch's version**.
- Everything between `=======` and `>>>>>>> feature/theme` is **the incoming branch's version**.
- You must manually edit the file to the desired final state and remove the markers.

---

### Common Causes of Frequent Conflicts

1. **Long-lived branches** — the longer a branch lives, the more it diverges from main. Every day you don't sync is another day of potential conflict accumulating.
2. **Large teams editing the same files** — poor code organization leads multiple developers to change the same "central" files constantly.
3. **Poor communication** — two devs working on the same feature area without coordinating.
4. **No clear file ownership** — when "everyone touches everything," conflicts are inevitable.
5. **Infrequent integration** — teams that merge weekly instead of daily accumulate massive diffs.

---

### How to Handle a Conflict

```bash
# 1. Attempt the merge (or rebase, or pull)
git merge feature/theme
# → CONFLICT (content): Merge conflict in style.css

# 2. Find all conflicted files
git status
# → both modified: style.css

# 3. Open each file and resolve manually
# Edit the file: delete the conflict markers, keep what you want

# 4. Stage the resolved file
git add style.css

# 5. Complete the merge
git commit
# (Git pre-fills a merge commit message)
```

Tools to help:
- `git mergetool` — opens a visual three-way diff tool (VSCode, vimdiff, etc.)
- VS Code has built-in conflict resolution UI with "Accept Current / Accept Incoming / Accept Both" buttons.

---

### Prevention Strategies

| Strategy | How it helps |
|---|---|
| **Short-lived branches** | Less time to diverge, fewer conflicts |
| **Frequent integration** | Merge to main (or rebase onto main) daily or every few commits |
| **Clear file ownership** | Each team/developer "owns" certain modules — reduces overlap |
| **Feature flags** | Merge incomplete features to main behind a flag — keeps branches short without exposing incomplete work |
| **Small, focused PRs** | Smaller changes = smaller diffs = fewer collision points |
| **Trunk-based development** | Everyone commits to main frequently — forces continuous integration and prevents long-lived divergence |
