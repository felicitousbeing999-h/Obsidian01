


> **Intent:** Build → Scan → Push → Deploy → Scale  
> **Reality:** Many rocks fell. Caveman fixed rocks. App live.

---

## 🎯 What Caveman Wanted

```mermaid
flowchart LR
    G[GitHub] --> AD[Azure DevOps]
    AD --> B[Build .NET 8]
    B --> D[Docker]
    D --> T[Trivy]
    T --> H[Docker Hub]
    H --> K[AKS]
    K --> U[User sees shop]
```

| Want | Tool |
|------|------|
| CI | Azure DevOps YAML |
| Images | Docker Hub (cheap) |
| Cluster | AKS 1 node B2s |
| DB | Mongo |
| Scale | HPA |
| Cost | Low |

**Not want:** Expensive ACR forever. Big cluster. Fancy GitOps yet.

---

## 🗺️ Final Map

```mermaid
flowchart TB
    subgraph SRC[Source]
        GH[GitHub Deployment branch]
    end

    subgraph CI[CI Pipeline]
        R[.NET Restore]
        BU[Build]
        DK[Docker Build]
        TR[Trivy soft-fail]
        PU[Push Docker Hub]
    end

    subgraph REG[Registry]
        DH[hardik0811/shoppingapi]
        DH2[hardik0811/shoppingclient]
    end

    subgraph AKS[AKS k8sgpt]
        NS[default ns]
        M[(Mongo)]
        API[shoppingapi]
        CLI[shoppingclient]
        LB[LoadBalancer]
    end

    GH --> CI
    CI --> DH & DH2
    DH --> API
    DH2 --> CLI
    M --> API
    API --> CLI
    CLI --> LB
    LB --> Human
```

---

## 💀 Failure Timeline

```mermaid
timeline
    title Caveman vs Cloud
    section Registry
        ACR cost high : Switch Docker Hub
    section Cluster
        AKS deleted for $ : Recreate 1 node
        Cloud Shell paste crash : File + --no-wait
        ResourceNotFound : Create never finished
    section Pipeline
        AKS-Connection missing : ARM service connection
        Trivy artifact / invalid : Rename trivy-report
        403 listClusterUserCredential : Role on SP
        kubelogin missing : useClusterAdmin true
    section Runtime
        JsonReaderException S : Mongo dead
        Mongo OOMKilled 128Mi : Bump 256/512
        kubectl Forbidden : --admin creds
        Wrong namespace : default not application
```

---

## 🔥 Failures → Fixes (Picture Book)

### 1. ACR too expensive

```mermaid
flowchart LR
    A[ACR $$] -->|migrate| B[Docker Hub free tier]
```

**Fix:** `DockerHub Registry Connection` + `hardik0811/...`

---

### 2. Cloud Shell dies on paste

```mermaid
sequenceDiagram
    Caveman->>Shell: paste long script
    Shell-->>Caveman: 💥 disconnect
    Caveman->>Shell: cat > setup.sh
    Caveman->>Shell: bash setup.sh --no-wait
    Shell-->>Caveman: cluster Creating...
```

**Fix:** Write file. `--no-wait`. Poll status.

---

### 3. Pipeline invalid — AKS-Connection

```mermaid
flowchart TD
    Y[YAML: azureResourceManager] --> Need[ARM service connection]
    K[Kubernetes connection] -.->|wrong type| Fail[not found]
    Need --> OK[AKS-Connection ARM]
```

**Fix:** Service connection type = **Azure Resource Manager**, name exact `AKS-Connection`.

---

### 4. Trivy artifact name

```text
BAD:  trivy-hardik0811/shoppingapi   ← slash illegal
GOOD: trivy-report
```

```mermaid
flowchart LR
    Img[hardik0811/shoppingapi] -->|used as artifact name| Boom[❌]
    Img --> Scan[trivy scan]
    Scan --> Art[artifact: trivy-report ✅]
```

---

### 5. 403 credentials

```mermaid
flowchart TD
    SP[Service Principal] -->|no role| Deny[403]
    SP -->|Cluster User Role| Creds[get kubeconfig]
    SP -->|RBAC Cluster Admin| Apply[kubectl apply]
```

**Fix:**

```bash
# on SP object id from error
Azure Kubernetes Service Cluster User Role
Azure Kubernetes Service RBAC Cluster Admin
Azure Kubernetes Service Cluster Admin Role   # for useClusterAdmin
```

---

### 6. kubelogin missing

```mermaid
flowchart LR
    AAD[AAD cluster] --> Plugin[needs kubelogin]
    Agent[Hosted agent] --> No[no kubelogin]
    Plugin --> Fail
    Fix[useClusterAdmin: true] --> OK
```

---

### 7. Json error on website

```mermaid
sequenceDiagram
    Browser->>Client: GET /
    Client->>API: GET /product
    API->>Mongo: query
    Mongo--xAPI: DEAD OOM
    API-->>Client: text starting with S
    Client-->>Browser: JsonReaderException 💥
```

**Root:**

| Thing | Status |
|-------|--------|
| Client | Running |
| API | Running |
| Mongo | **OOMKilled** limit 128Mi |

**Fix:** Mongo `256Mi` request / `512Mi` limit.

```mermaid
flowchart LR
    Before[128Mi] -->|OOMKill| Dead
    After[512Mi] -->|alive| JSON
    JSON --> Happy[Shop works]
```

---

### 8. kubectl Forbidden

```mermaid
flowchart LR
    User -->|Azure RBAC lag| Forbidden
    User -->|get-credentials --admin| Works
```

**Fix:** `az aks get-credentials ... --admin`

---

## 🧬 Identity of Images (sacred rule)

```mermaid
flowchart TD
    Build[Build name] --> Same
    Scan[Scan name] --> Same
    Push[Push name] --> Same
    Same[hardik0811/shoppingapi:BuildId]
```

Break one → tag mismatch → push/pull fail.

---

## 📦 Repo Shape

```text
pipelines/
  shoppingapi-pipeline.yaml
  shoppingclient-pipeline.yaml
  templates/
    common/variables.yml
    ci/build-dotnet.yml
    security/trivy-scan.yml
    cd/deploy-infra.yml
    cd/deploy-aks.yml
k8s/   ← what CD deploys
aks/   ← lab manifests / LB / HPA
Shopping/
  Shopping.API/
  Shopping.Client/
```

---

##  variables.yml 

| Key | Value |
|-----|--------|
| dockerHubUsername | hardik0811 |
| dockerRegistryServiceConnection | DockerHub Registry Connection |
| aksServiceConnection | AKS-Connection |
| aksResourceGroup | k8sgpt |
| aksClusterName | ASP-MicroserviceApplication |
| aksNamespace | application *(runtime often default)* |

---

##  Pipeline Stages

```mermaid
stateDiagram-v2
    [*] --> CI
    CI --> Build
    Build --> Trivy
    Trivy --> Push
    Push --> CD: branch main OR Deployment
    CI --> [*]: failed
    CD --> Infra: mongo + configmaps
    Infra --> Apps: api + client
    Apps --> [*]
```

CD skipped when CI fails (`succeeded()` gate).

---

## 🧪 Live Checks (caveman commands)

```bash
# cluster
az aks show -g k8sgpt -n ASP-MicroserviceApplication --query provisioningState -o tsv

# auth that works
az aks get-credentials -g k8sgpt -n ASP-MicroserviceApplication --admin

# health
kubectl get pods
kubectl get svc
kubectl logs deploy/mongo-deployment --tail=30
kubectl logs deploy/shoppingapi-deployment --tail=30

# API from inside
kubectl exec deploy/shoppingclient-deployment -- wget -qO- http://shoppingapi-service:8000/product

# HPA watch
kubectl get hpa,pods -w
```

---

## 📈 HPA Demo Flow

```mermaid
flowchart LR
    Curl[curl storm] --> CPU[CPU up]
    CPU --> HPA[HPA sees %]
    HPA --> More[more pods]
    Stop[stop curl] --> Wait[3-5 min]
    Wait --> Less[scale down]
```

**Warn:** `targetCPU: 2%` = scale always. 1 node + minReplicas 2+3 = tight.

---

## 🧠 Lessons Carved in Stone

```mermaid
mindmap
  root((Lessons))
    Cost
      ACR optional
      Docker Hub ok for lab
    Pipeline
      Artifact names no slash
      Connection type must match YAML
      SP needs Azure roles
    AKS
      AAD needs kubelogin or admin
      RBAC lag is real
    Workloads
      Mongo needs real memory
      Client dies if API body not JSON
      Namespace must match everywhere
```

---

## ✅ Done vs Next

| Done | Next |
|------|------|
| CI build + Trivy + push | Hard fail on CRITICAL |
| CD to AKS | Force `application` ns |
| Docker Hub | Optional imagePullSecret |
| Mongo not OOM | Pin `mongo:6` if still heavy |
| LB frontend | Ingress + TLS |
| HPA manifests | Real load test script |
| | Terraform for AKS |

---

## 🏁 One Picture End State

```mermaid
flowchart TB
    Human((Human)) -->|http://LB-IP| Client
    Client -->|/product JSON| API
    API --> Mongo[(Mongo 512Mi)]
    
    subgraph Azure DevOps
        Pipe[CI/CD YAML]
    end
    
    subgraph Docker Hub
        I1[shoppingapi]
        I2[shoppingclient]
    end
    
    Pipe --> I1 & I2
    I1 --> API
    I2 --> Client
```

**Caveman goal:** Artifact walks path  
`code → image → scan → registry → cluster → browser`  
without mystery failures.

**Caveman status:** Path works. Mongo fed. Rocks documented.

