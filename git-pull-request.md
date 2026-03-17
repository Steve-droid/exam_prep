# Pull Request

## Question
What is a pull request? When to use it, difference between `git push` and a PR, and PR merge types.

## Ideal One-Liner
A pull request is a request to merge your branch into a target branch on the remote, with a built-in code review and discussion layer before the merge happens.

---

## Full Answer

**When to use:** whenever merging to a shared branch (main, develop) — enables review, CI checks, and an audit trail.

**`git push` vs PR:** `git push` uploads commits to the remote. A PR is a *proposal* to merge those commits into another branch — it adds review, approval, and CI gating on top.

**Merge types:**

| Type         | What happens                                            | When to use                          |
| ------------ | ------------------------------------------------------- | ------------------------------------ |
| Merge commit | Creates a new merge commit with two parents             | Preserving full branch history       |
| Squash       | Collapses all PR commits into one commit on target      | Clean history, small features/fixes  |
| Rebase       | Replays commits on top of target, new SHAs (D', E', F') | Linear history without merge commits |

---

## Q&A

**Q: In a rebase PR, who actually runs `git rebase`?**
Nobody manually runs it. The platform (GitHub/GitLab) runs it server-side when the PR is merged. The developer clicks "Rebase and merge" and GitHub replays the commits on top of the target branch, creating new commit SHAs. The dev never runs `git rebase` locally for this.

**Q: Who chooses the merge type — the dev or the approver?**
Whoever clicks the merge button — usually the reviewer/approver. On GitHub, there's a dropdown with three options: "Create a merge commit", "Squash and merge", "Rebase and merge". Repo admins can restrict which options are available in repo settings.
