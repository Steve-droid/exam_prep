# Handling Accidentally Committed Sensitive Data

## Question
You accidentally committed sensitive data (like an API key) and pushed it. How do you handle this?

## Ideal One-Liner
Rotate the secret immediately — assume it's already compromised — then remove it from history using BFG or git filter-branch, force-push, and coordinate the team to re-clone or reset.

---

## Full Answer

### Step 1: Rotate the Secret Immediately (Do This First)

The moment a secret is pushed to a remote repository, **assume it is compromised.** Even if the repo is private:
- Someone may have already cloned or forked it.
- GitHub/GitLab may have cached the content.
- CI/CD systems may have already logged it.

**Before doing anything else:**
- Revoke/rotate the API key, token, password, or certificate.
- Generate a new secret.
- Update it in your application's secrets manager or environment variables.

Removing the secret from history is important, but it's secondary to revoking it.

---

### Step 2: Remove From the Latest Commit (If Just Pushed)

#### If the bad commit is the most recent one:

```bash
# Remove the file with the secret from tracking
echo "secrets.env" >> .gitignore
git rm --cached secrets.env

# Amend the commit (rewrites it — only safe if you act immediately and no one has pulled)
git commit --amend --no-edit

# Force push
git push --force-with-lease origin main
```

Or reset if you want to stage fresh:
```bash
git reset HEAD~1          # unstages the last commit
# remove the secret from the file or delete the file
git add .
git commit -m "Add feature without secrets"
git push --force-with-lease origin main
```

---

### Step 3: Remove From Full Git History

If the secret appeared in an older commit, you must **rewrite the entire history** to remove it. Use one of:

#### Option A: BFG Repo-Cleaner (recommended — simpler)
```bash
# Download BFG: https://rtyley.github.io/bfg-repo-cleaner/
java -jar bfg.jar --delete-files secrets.env my-repo.git
cd my-repo
git reflog expire --expire=now --all
git gc --prune=now --aggressive
git push --force origin main
```

Or to replace specific strings (e.g., the API key value):
```bash
echo "SUPER_SECRET_KEY_VALUE" > secrets.txt
java -jar bfg.jar --replace-text secrets.txt my-repo.git
```

#### Option B: git filter-branch (built-in but slower)
```bash
git filter-branch --force --index-filter \
  'git rm --cached --ignore-unmatch secrets.env' \
  --prune-empty --tag-name-filter cat -- --all

git push --force origin main
```

---

### Step 4: Team Coordination After Force-Push

Rewriting history means everyone who cloned the repo has a diverged history. All team members must:

```bash
# Option 1: re-clone the repo
git clone <repo-url>

# Option 2: hard reset to the remote state
git fetch origin
git reset --hard origin/main
```

**Communicate clearly** before forcing: tell the team what happened, tell them to stop pushing, give them the reset steps.

---

### Step 5: GitHub/GitLab Assistance

Even after rewriting history, some platforms cache commit content:
- **GitHub:** Contact GitHub Support — they can force-purge cached views of the old commits.
- **GitLab:** Similar support request, plus check your CI/CD job logs for the secret.

Also check:
- Are there any **forks** of the repo? They retain the old history.
- Any **PRs** that show the old diff? Request they be closed.
- Any **CI/CD logs** that printed the secret value? Purge those too.

---

### Prevention

- Use a `.gitignore` with patterns for `.env`, `*.key`, `*credentials*`.
- Use pre-commit hooks: `git-secrets`, `gitleaks`, or `detect-secrets` to scan before committing.
- Store secrets in a secrets manager (AWS Secrets Manager, Vault, GitHub Secrets) — never in code.
- Follow the principle: if it's a secret, it never belongs in source control.
