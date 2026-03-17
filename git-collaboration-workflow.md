# Git Collaboration Workflow

## Question
Describe how several people should use Git to collaborate on the same project.

## Ideal One-Liner
Each developer works on a short-lived feature branch off main, submits a pull request for code review, and merges only when CI passes — keeping main always deployable.

---

## Full Answer

### Branching Strategy

```
main:     A ── B ────────────── F ── G ──► (always deployable)
               │               ↑
feature/auth:  └── C ── D ──── ┘  (PR merged after review)
               │
feature/api:   └── E ────────────── (still in progress)
```

Rules:
- **main is sacred** — direct commits are forbidden (enforced via branch protection rules).
- Every piece of work starts on a **named feature branch**: `feature/user-auth`, `bugfix/null-pointer`, etc.
- Branches are **short-lived** — days, not weeks. The longer a branch lives, the more it diverges from main and the worse the merge conflict.

---

### Pull Requests (PRs)

A PR is a formal request to merge your branch into main. It serves multiple purposes:

1. **Code review** — at least one other developer reads and approves the changes.
2. **CI gate** — automated tests must pass before merge is allowed.
3. **Discussion thread** — comments, suggestions, and change requests are tracked in the PR.
4. **Audit trail** — the PR records *why* the change was made, not just what.

**PR merge options:**
| Type | What it does | When to use |
|---|---|---|
| Merge commit (`--no-ff`) | Preserves branch history with a merge commit | Default; shows feature groupings |
| Squash and merge | Collapses all branch commits into one on main | Messy commit history on branch, want clean main |
| Rebase and merge | Replays commits linearly onto main | Want linear history without a merge commit |

---

### Fast-Forward and Why It Matters
If a feature branch hasn't diverged from main (main hasn't moved), the merge is a **fast-forward** — the branch pointer just moves forward, no merge commit needed. This keeps history linear and readable. To enable fast-forward, developers should regularly rebase their branches onto main.

---

### CI Differences by Branch Type

| Branch | CI pipeline runs |
|---|---|
| `feature/*` | Unit tests, linting — fast feedback, catches obvious breakage |
| `main` | Full integration tests, security scans, build artifact |
| `release/*` | Integration tests + deployment to staging, smoke tests |

This mirrors what was built in Gan Shmuel: the CI service would detect a push, run tests in a test environment, and only promote to prod if tests passed.

---

### Team Workflow Step by Step

```bash
# 1. Start from up-to-date main
git checkout main
git pull origin main

# 2. Create feature branch
git checkout -b feature/my-feature

# 3. Do work, commit regularly
git add .
git commit -m "Add X functionality"

# 4. Keep branch updated (rebase preferred for solo branches)
git fetch origin
git rebase origin/main

# 5. Push and open PR
git push origin feature/my-feature
# → open PR on GitHub/GitLab

# 6. After approval and CI pass: merge via UI
# 7. Delete feature branch after merge
```

---

### Practical Rules That Prevent Pain

- **Never commit directly to main** — always use a branch + PR.
- **Keep branches short-lived** — merge or discard within a few days.
- **Pull/rebase often** — sync with main regularly to avoid massive divergence.
- **Write meaningful commit messages** — `git log` should tell a story.
- **One logical change per PR** — easier to review, easier to revert if needed.

---

### Gan Shmuel Reference
In Gan Shmuel, the CI pipeline (`ci/app.py`) was triggered by a GitHub webhook on push to main. The team worked on feature branches and merged via PRs. The CI service then ran the full pipeline: git pull → build → test deploy (port 8082/8083) → pytest → prod deploy (8080/8081) → email notification. This is the exact collaboration workflow described above, implemented in code.
