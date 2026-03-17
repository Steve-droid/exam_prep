# Git Branching Strategies

## Question
Compare trunk-based development, Git Flow, and the Mainline Model. When would you use each?

## Ideal One-Liner
Trunk-based development prioritizes continuous integration on a single main branch; Git Flow adds structured release and hotfix branches for versioned releases; the Mainline Model maintains long-lived release branches to support multiple deployed versions simultaneously.

---

## Full Answer

---

### 1. Trunk-Based Development

**Core idea:** Everyone integrates into `main` (the "trunk") frequently — at least daily. Feature branches exist but are very short-lived (hours to 1–2 days max).

```
main:   A ── B ── C ── D ── E ── F ──► continuous deployment
             ↑         ↑
         (feat/x)   (feat/y)
         merged      merged
         same day    next day
```

**Key characteristics:**
- No long-lived branches except main.
- Incomplete features are merged behind **feature flags** (runtime toggles).
- Requires strong CI/CD — every commit to main should be deployable.
- Deployment happens continuously (SaaS model).

**When to use:**
- SaaS products with continuous deployment.
- Teams with strong CI/CD discipline.
- High-frequency release cadence (multiple times per day/week).

**Trade-offs:**
- Simple branch structure, easy to manage.
- Requires discipline: feature flags, thorough automated tests, mature CI/CD.
- Not suitable if you need to support multiple released versions.

---

### 2. Git Flow

**Core idea:** Structured workflow with dedicated branches for each phase of the release cycle.

```
main:    ──────────────────────────────────────► (only tagged releases)
            ↑                     ↑
develop: A ── B ── C ── D ── E ── F ── G ──►    (integration branch)
              ↑         ↑
         feature/x   feature/y
         (merged)     (merged)

release/1.0:      ──── R1 ── R2 ──►  (branched from develop, merged to main + develop)

hotfix/1.0.1:                  H ──► (branched from main, merged to main + develop)
```

**Branch types:**
| Branch | Purpose |
|---|---|
| `main` | Only tagged, production-ready releases |
| `develop` | Integration branch, all features merge here |
| `feature/*` | Individual features, branch from develop |
| `release/*` | Release preparation (bugfixes, version bumps), no new features |
| `hotfix/*` | Critical production fixes, branch from main |

**When to use:**
- Products with scheduled, versioned releases (e.g., mobile apps, packaged software).
- Teams with defined QA/release cycles.
- When you need the structure and formality.

**Trade-offs:**
- Structured and explicit — everyone knows what goes where.
- **Risk: long-lived branches** — `develop` and `release` can diverge significantly from `main`, leading to painful merge conflicts.
- Overhead: maintaining multiple branches, remembering merge rules.

---

### 3. Mainline Model

**Core idea:** Developers work on `main`. Long-lived **release branches** are cut for each major version and maintained independently. Critical fixes are backported from main to affected release branches.

```
main:      A ── B ── C ── D ── E ── F ──► (ongoing development)
                │                   │
release/v1.x:  └── v1.0 ── v1.1 ── v1.2   ← maintained by release manager
               │
release/v2.x:      └── v2.0 ── v2.1        ← separate release branch
```

**How fixes work:**
1. Developer commits fix to `main`.
2. Fix is cherry-picked to `release/v1.x` and `release/v2.x` if they're still supported.
3. Release manager tags a new patch version on the release branch.

**When to use:**
- On-premises software shipped to enterprise clients.
- Products where customers are on different versions and can't always upgrade.
- Open-source projects with LTS releases (e.g., Node.js, Ubuntu LTS).

**Trade-offs:**
- Supports multiple live versions cleanly.
- Cherry-picking fixes requires discipline and coordination.
- More branches to maintain than trunk-based, but less ceremony than Git Flow.

---

### Decision Guide

| Factor | Trunk-Based | Git Flow | Mainline |
|---|---|---|---|
| **Team size** | Any (with CI) | Medium–large | Any |
| **Deployment freq.** | Continuous | Scheduled releases | Per-version |
| **Multiple versions in prod?** | No | No | Yes |
| **CI/CD maturity needed** | High | Medium | Medium |
| **Branch complexity** | Low | High | Medium |
| **Best for** | SaaS, startups | Packaged software | Enterprise/on-prem |
