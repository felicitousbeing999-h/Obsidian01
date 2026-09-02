![[Pasted image 20260902125530.png]]
> [!abstract] Pattern  
> **Deployment → Pod → InitContainer → Shared Volume → Main Container**

---

## 1. Architecture

```mermaid
flowchart TB
    D["Deployment<br/>ic-deploy-xfusion<br/>replicas: 1"]
    RS["ReplicaSet"]
    P["Pod<br/>app: ic-xfusion"]

    D --> RS
    RS --> P

    P --> I["InitContainer<br/>ic-msg-xfusion<br/>debian:latest"]
    P --> M["Main Container<br/>ic-main-xfusion<br/>debian:latest"]

    I --> V[("emptyDir<br/>ic-volume-xfusion")]
    V --> M

    I -->|"writes /ic/blog"| V
    M -->|"reads /ic/blog"| V
```

---

## 2. Deployment Selector

### Critical invariant

```mermaid
flowchart LR
    S["Deployment Selector<br/>app: ic-xfusion"]
    L["Pod Template Label<br/>app: ic-xfusion"]

    S ===|"MUST MATCH"| L

    S -->|"controls"| P["Pods"]
```

```yaml
selector:
  matchLabels:
    app: ic-xfusion

template:
  metadata:
    labels:
      app: ic-xfusion
```

> [!danger] Production rule  
> `spec.selector.matchLabels` is the Deployment's **identity contract** for its Pods.

---

## 3. InitContainer Lifecycle

```mermaid
sequenceDiagram
    participant K as Kubelet
    participant I as InitContainer
    participant V as emptyDir
    participant M as Main Container

    K->>I: Start
    I->>V: Write /ic/blog
    I-->>K: Exit 0
    K->>M: Start
    M->>V: cat /ic/blog
    M-->>M: sleep 5
    M->>V: cat /ic/blog
```

### Key property

```text
InitContainer
     │
     │ MUST succeed
     ▼
Main Container starts
```

```mermaid
flowchart LR
    A["InitContainer"] --> B{"Exit 0?"}
    B -->|Yes| C["Start Main Container"]
    B -->|No| A
```

---

## 4. Shared `emptyDir`

```mermaid
flowchart TB
    POD["Pod"]

    I["InitContainer"]
    M["Main Container"]

    V[("emptyDir<br/>/ic")]

    POD --> I
    POD --> M

    I -->|"mount /ic"| V
    M -->|"mount /ic"| V
```

### Storage lifecycle

```mermaid
flowchart LR
    P1["Pod created"] --> V["emptyDir created"]
    V --> W["Containers share data"]
    W --> D["Pod deleted"]
    D --> X["emptyDir deleted"]
```

> [!warning] Architect's rule  
> `emptyDir` is **Pod-scoped ephemeral storage**.
> 
> **Pod dies → data dies.**

---

# 5. Container Responsibilities

|Component|Responsibility|
|---|---|
|`ic-msg-xfusion`|Initialization|
|`ic-volume-xfusion`|Temporary shared filesystem|
|`ic-main-xfusion`|Long-running workload|

```mermaid
flowchart LR
    I["Init<br/>Prepare"]
    V["Shared<br/>State"]
    M["Application<br/>Consume"]

    I --> V --> M
```

---

# 6. Why InitContainers Exist

```mermaid
flowchart TB
    A["Pod starts"]

    A --> B["Initialization"]
    B --> C["Validation / Migration / Preparation"]
    C --> D["Application"]

    style B fill:transparent
    style C fill:transparent
```

Typical responsibilities:

```text
InitContainer
├── generate configuration
├── download/bootstrap files
├── database migration
├── permissions / filesystem setup
├── wait for dependency
└── service initialization
```

---

# 7. Production Workload Example

## Configuration bootstrap for a web application

A realistic production pattern:

```mermaid
flowchart TB
    P["Application Pod"]

    I["InitContainer<br/>config-renderer"]
    V[("emptyDir<br/>/config")]
    A["Application Container"]

    I -->|"render config"| V
    V -->|"read-only mount"| A
```

### Example workload

```text
InitContainer
     │
     ├── fetch configuration
     ├── render application.conf
     └── validate configuration
              │
              ▼
        /config/app.conf
              │
              ▼
Application Container
```

**When this is useful in production:**

> A platform team can keep application containers immutable while an InitContainer prepares runtime configuration before the application starts.

This pattern appears in workloads involving **configuration generation, certificate/bootstrap preparation, migrations, dependency checks, and sidecar-style initialization**.

---

# 8. `emptyDir` vs PersistentVolume

```mermaid
flowchart TB
    STORAGE["Storage"]

    STORAGE --> E["emptyDir"]
    STORAGE --> P["PersistentVolume"]

    E --> E1["Pod lifetime"]
    E --> E2["Temporary data"]
    E --> E3["Cache / generated files"]

    P --> P1["Persistent"]
    P --> P2["Database / durable data"]
    P --> P3["Survives Pod recreation"]
```

### Decision

```text
Need data after Pod deletion?
        │
   ┌────┴────┐
   │         │
  YES       NO
   │         │
  PV      emptyDir
```

---

# 9. Validation Commands

```bash
kubectl get deployment ic-deploy-xfusion

kubectl get pods

kubectl describe pod <pod-name>

kubectl logs <pod-name> -c ic-msg-xfusion

kubectl logs <pod-name> -c ic-main-xfusion
```

Expected:

```text
Deployment
    │
    └── replicas: 1

Pod
    │
    └── 1/1 Running

InitContainer
    │
    └── Completed

MainContainer
    │
    └── Running
```

---

# 10. Debugging Mental Model

```mermaid
flowchart TB
    A["Deployment invalid?"]
    A --> B["Check selector ↔ labels"]

    B --> C["Pod not starting?"]
    C --> D["Check InitContainer"]

    D --> E["Init succeeds?"]
    E -->|No| F["Inspect init logs/events"]
    E -->|Yes| G["Main starts"]

    G --> H["Shared data missing?"]
    H --> I["Check volume + mounts"]

    I --> J["emptyDir mounted at same path?"]
```

---

# 11. Final Architecture Principle

```mermaid
flowchart LR
    DEP["Deployment"]
    SEL["Selector"]
    POD["Pod"]
    INIT["InitContainer"]
    VOL["emptyDir"]
    APP["Application"]

    DEP --> SEL
    SEL -->|"app=ic-xfusion"| POD
    POD --> INIT
    INIT -->|"prepare"| VOL
    VOL -->|"consume"| APP
```

> [!tip] Remember  
> **Selector identifies → Init prepares → Volume shares → Main consumes**

#kubernetes #deployment #initcontainer #emptydir #volumes #platform-engineering #cloud-architecture