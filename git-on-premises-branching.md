# Branching Strategy for On-Premises Enterprise Software

## Question
Your company ships on-premises software to enterprise clients on different versions. Which branching strategy would you recommend and why?

## Ideal One-Liner
Use the Mainline Model: developers work on main, and long-lived release branches (v1.x, v2.x) are maintained by release managers to support each deployed version independently.

---

## Full Answer

### Why This Scenario Is Different

Enterprise clients running on-premises software:
- Cannot or will not upgrade immediately when a new version ships.
- May be 1–2 major versions behind.
- Still need **security patches and critical bugfixes** on the version they're running.
- New features go into the next major version, not into older deployed versions.

This means you need to be able to **release independently on multiple version lines** — something trunk-based development and Git Flow don't handle cleanly.

---

### The Mainline Model

```
main (development):
A ── B ── C ── D ── E ── F ── G ──► (new features, ongoing work)
     │                   │
     │                   │
release/v1.x:            │
     └── v1.0 ── v1.1 ── v1.2 ── v1.3 ──► (supported, active)
                         │
release/v2.x:            └── v2.0 ── v2.1 ──► (newer major version)
```

---

### How It Works in Practice

**Normal feature development:**
- Developers commit to `main` (or short-lived feature branches off main).
- `main` always represents the next, unreleased version.

**Cutting a release branch:**
- When a major version is ready to ship (e.g., v2.0), a release manager cuts `release/v2.x` from main.
- From that point on, `release/v2.x` is independent.
- No new features go into the release branch — only bugfixes and security patches.

**Backporting a critical fix:**
```bash
# 1. Developer commits fix to main
git checkout main
git commit -m "Fix: SQL injection in user query"

# 2. Release manager identifies which release branches are affected
# 3. Cherry-pick the fix to each active release branch
git checkout release/v1.x
git cherry-pick <commit-sha>
git tag v1.4.1

git checkout release/v2.x
git cherry-pick <commit-sha>
git tag v2.1.1
```

---

### Role of Release Managers
In this model, **release branches are owned by release managers**, not developers:
- Developers work on `main` — they don't commit to release branches directly.
- Release managers decide which fixes from `main` get backported.
- Release managers are responsible for tagging, packaging, and distributing each release.
- This separation keeps release branches stable and prevents accidental regressions.

---

### Why Not Git Flow?
Git Flow has a `release` branch too, but it's designed for a **single active release line**. Once a release branch is merged to main and deleted, it's gone. Git Flow doesn't have a clean mechanism to maintain `v1.x` and `v2.x` simultaneously over months or years.

### Why Not Trunk-Based?
Trunk-based has no concept of long-lived release lines. It assumes continuous deployment to a single production environment — not viable when clients run your software on their own infrastructure.

---

### Summary

| Aspect | Mainline Model Behavior |
|---|---|
| Where devs work | `main` |
| Release per version | Long-lived `release/vX.x` branch |
| Critical fixes | Committed to `main`, cherry-picked to affected release branches |
| Who manages releases | Dedicated release managers |
| New features in old releases? | No — only bugfixes/security patches |
| Number of concurrent versions | Unlimited — each has its own branch |
