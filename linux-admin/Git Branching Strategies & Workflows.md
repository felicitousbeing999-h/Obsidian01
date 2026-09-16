#azure-devops 

> [!summary] Rule  
> **Branch → Work → PR → Check → Merge → `main`**
> 
> Keep `main` healthy. Keep branches short-lived.

---

## 1. Basic model

```mermaid
flowchart LR
    MAIN[(main)] --> FEAT[feature/*]
    FEAT --> PR[Pull Request]
    PR --> CHECK[Review + CI + Security]
    CHECK -->|Pass| MAIN
    CHECK -->|Fail| FEAT
```

```text
main = stable
feature = work
PR = review
CI = test
merge = integrate
```

Microsoft baseline:

```text
Feature branches
      ↓
Pull Requests
      ↓
High-quality main
```

---

# 2. Feature Branch Workflow

**One feature/bug = one branch.**

```mermaid
gitGraph
    commit id: "main"
    branch feature/login
    checkout feature/login
    commit id: "login"
    commit id: "test"
    checkout main
    merge feature/login id: "PR"
```

```text
main
 ├── feature/login
 ├── feature/payment
 └── bugfix/api
          ↓
         PR
          ↓
        main
```

### Why?

```text
No branch:
dev → main → chaos

Branches:
dev → feature → PR → main
```

---

# 3. Keep Feature Branch Current

Main moves:

```text
main:     A──B──C──D
               \
feature:         F──G
```

Rebase:

```text
A──B──C──D──F'──G'
```

```bash
git fetch origin
git rebase origin/main
```

**Rebase = replay my work on latest main.**

**Do not rebase shared/public work casually.**

---

# 4. Merge vs Rebase

```text
MERGE
A──B──C────M
   \      /
    F────G

REBASE
A──B──C──F'──G'
```

||Meaning|
|---|---|
|`merge`|Join histories|
|`rebase`|Replay commits on new base|
|`PR`|Review + controlled merge|

---

# 5. GitHub Flow

**Simple + frequent delivery.**

```mermaid
flowchart LR
    MAIN[(main)] --> FEATURE[feature]
    FEATURE --> PR[PR]
    PR --> CI[CI]
    CI --> MAIN
    MAIN --> DEPLOY[Deploy]
```

Use when:

```text
frequent releases
+
continuous delivery
+
short-lived branches
+
PRs
```

**Main should stay deployable.**

---

# 6. Release Branch

**Problem: release needs stabilization while new work continues.**

```mermaid
flowchart LR
    MAIN[(main)] --> REL[release/2.0]
    MAIN --> NEW[New Features]

    REL --> FIX[Release Fixes]
    FIX --> TEST[Test]
    TEST --> PROD[Production]
```

```text
main
 ├── future work
 │
 └── release/2.0
       ├── fix
       ├── test
       └── production
```

Use when:

```text
Need release stabilization
        ↓
release/*
```

Not:

```text
Compliance = release branch
```

---

# 7. Compliance

**Branch type does not create compliance. Controls do.**

```mermaid
flowchart LR
    PR[PR] --> REVIEW[Review]
    PR --> TEST[Test]
    PR --> SEC[Security]
    PR --> BUILD[Build]

    REVIEW --> GATE{Gate}
    TEST --> GATE
    SEC --> GATE
    BUILD --> GATE

    GATE -->|Pass| MERGE[Merge]
    GATE -->|Fail| BLOCK[Block]
```

Typical:

```text
Protected main
PR required
Reviewer required
Build required
Tests required
Security checks
Audit history
No direct push
```

**Compliance = controls around change.**

---

# 8. Hotfix

Problem:

```text
production = old stable version
main       = new + unfinished work
```

Do this:

```mermaid
flowchart LR
    STABLE[(Stable Release)] --> HOTFIX[hotfix/*]
    HOTFIX --> PROD[Production]
    HOTFIX --> MAIN[(main)]
```

```text
Stable
  ↓
Hotfix
  ├──→ Production
  └──→ main
```

**Fix production version. Then carry fix forward.**

---

# 9. Forking Workflow

**Fork = separate repository copy.**

```mermaid
flowchart TB
    UP[(Upstream)]
    UP --> A[Fork A]
    UP --> B[Fork B]

    A --> PR1[PR]
    B --> PR2[PR]

    PR1 --> UP
    PR2 --> UP
```

Use for:

```text
Open source
External contributors
No direct write access
Distributed contributors
```

**Fork ≠ feature branch.**

---

# 10. Release Fix → main

Release branch has:

```text
release
 ├── release-only change
 └── important bug fix
```

Need only bug fix in `main`:

```mermaid
flowchart LR
    REL[release/*] --> FIX[Bug Fix]
    FIX --> PROD[Release]
    FIX --> CP[Cherry-pick]
    CP --> MAIN[(main)]
```

**Cherry-pick = take this exact commit.**

Useful when:

```text
Release fix
   ↓
Need same fix in main
```

---

# 11. Continuous Deployment

Don't make:

```text
main
 ├── dev
 ├── staging
 └── production
```

Prefer:

```mermaid
flowchart LR
    MAIN[(main)] --> BUILD[Build Once]
    BUILD --> ART[Artifact]
    ART --> DEV[Dev]
    ART --> STAGE[Staging]
    ART --> PROD[Production]
```

**Build once → promote artifact.**

Use environment/deployment branches only when there is a real reason.

---

# 12. Decision Tree

```mermaid
flowchart TD
    Q{Problem?}

    Q -->|Normal feature work| F[Feature Branch]
    Q -->|Frequent CD| G[GitHub Flow]
    Q -->|Release stabilization| R[Release Branch]
    Q -->|External contributors| K[Fork Workflow]
```

---

# 13. Enterprise DevSecOps

```mermaid
flowchart LR
    DEV[Developer] --> F[feature/*]
    F --> PR[PR]

    PR --> REVIEW[Review]
    PR --> TEST[Test]
    PR --> SEC[Security]
    PR --> BUILD[Build]

    REVIEW --> G{Gate}
    TEST --> G
    SEC --> G
    BUILD --> G

    G -->|PASS| MAIN[(main)]
    G -->|FAIL| F

    MAIN --> ART[Artifact]
    ART --> DEPLOY[Progressive Deploy]
```

```text
feature
   ↓
PR
   ↓
review + test + security + build
   ↓
gate
   ↓
main
   ↓
artifact
   ↓
deploy
```

---

# 14. Cheat Sheet

|If question says...|Answer|
|---|---|
|Isolate feature|**Feature branch**|
|Code review|**PR**|
|Keep main stable|**Feature branch + PR**|
|Frequent CD|**GitHub Flow**|
|Stabilize release|**Release branch**|
|External contributors|**Fork**|
|Prevent direct push|**Branch policy**|
|Automated quality gate|**PR + CI policies**|
|Production bug + unfinished main|**Hotfix from stable release**|
|Release fix → main|**Cherry-pick**|
|Keep feature current|**Rebase / merge main**|
|CD environments|**Promote artifact**|

---

# 15. One Thing To Remember

```text
              MAIN
               │
        ┌──────┴──────┐
        ↓             ↓
     feature        bugfix
        │             │
        └──────┬──────┘
               ↓
              PR
               ↓
       REVIEW + CI + SECURITY
               ↓
             GATE
               ↓
          ┌────┴────┐
          │         │
        FAIL       PASS
          │         │
          ↓         ↓
        Fix       main
                    │
                    ↓
                 Artifact
                    │
                    ↓
                 Deploy
```

> **Caveman rule:**  
> **Don't work on `main`. Branch it. Review it. Test it. Protect it. Merge it.**