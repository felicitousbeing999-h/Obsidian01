![[Pasted image 20260902135450.png]]![[Pasted image 20260902140322.png]]
> [!abstract] Pattern  
> **Kubernetes Secret → Pod Volume → Container Filesystem**

---

## Architecture

```mermaid
flowchart LR
    F["/opt/beta.txt<br/>password / license"] 
    -->|kubectl create secret| S["Secret: beta"]

    S --> V["Secret Volume<br/>secret-volume"]

    V --> M["/opt/demo"]

    M --> C["secret-container-xfusion<br/>fedora:latest"]
```

---

## 1. Create Secret

```bash
kubectl create secret generic beta \
  --from-file=beta.txt=/opt/beta.txt
```

```mermaid
flowchart LR
    A["/opt/beta.txt"] --> B["Secret: beta"]
    B --> C["Key: beta.txt"]
```

---

## 2. Pod Manifest

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: secret-xfusion

spec:
  containers:
    - name: secret-container-xfusion
      image: fedora:latest
      command: ["sleep", "infinity"]

      volumeMounts:
        - name: secret-volume
          mountPath: /opt/demo
          readOnly: true

  volumes:
    - name: secret-volume
      secret:
        secretName: beta
```

### Critical Mapping

```mermaid
flowchart TD
    S["Secret: beta"]
    --> V["volume<br/>secret-volume"]
    --> VM["volumeMount"]
    --> P["/opt/demo"]

    K["beta.txt"]
    --> F["/opt/demo/beta.txt"]
```

> [!warning] YAML is case-sensitive  
> `mountPath` ✅  
> `MountPath` ❌

---

## 3. Deploy

```bash
kubectl apply -f pod.yaml
```

```bash
kubectl get pod secret-xfusion
```

Expected:

```text
secret-xfusion   1/1   Running
```

---

## 4. Verify

```bash
kubectl exec -it secret-xfusion \
  -c secret-container-xfusion -- bash
```

```bash
ls -l /opt/demo
cat /opt/demo/beta.txt
```

```mermaid
sequenceDiagram
    participant K as Kubernetes
    participant S as Secret
    participant P as Pod
    participant C as Container

    K->>S: Store beta
    K->>P: Create Pod
    P->>S: Mount Secret
    S-->>C: /opt/demo/beta.txt
    C->>C: Read secret file
```

---

# Production Pattern

### Typical workload

```mermaid
flowchart LR
    APP["Application Pod"]
    --> CFG["ConfigMap"]

    APP --> SEC["Secret"]
    SEC --> DB["DB Credentials"]

    APP --> TLS["TLS Secret"]
    TLS --> ING["Ingress / Gateway"]
```

### Real-world usage

|Secret|Typical use|
|---|---|
|DB password|PostgreSQL / Cloud SQL|
|API key|External SaaS API|
|TLS certificate|Ingress / Gateway|
|License key|Commercial software|
|Service credentials|Internal services|

---

## GKE Production Architecture

```mermaid
flowchart TB
    subgraph GKE["Google Kubernetes Engine"]
        POD["Application Pod"]
        
        POD --> SM["Secret Manager integration"]
        POD --> CSI["Secrets Store CSI Driver"]
    end

    SM["Google Secret Manager"]
    -->|secret retrieval| CSI

    CSI -->|mounted file| POD
```

> [!tip] Architecture decision  
> **Lab:** Kubernetes Secret volume  
> **Production GKE:** Prefer **Google Secret Manager + Secret Manager integration/CSI** for centrally managed sensitive data.

---

## Mental Model

```mermaid
flowchart LR
    SECRET["Secret"]
    --> VOLUME["Volume"]
    --> MOUNT["mountPath"]
    --> FILE["File inside container"]

    style SECRET stroke-width:3px
    style FILE stroke-width:3px
```

**Secret ≠ environment variable ≠ volume**

The application simply reads:

```text
/opt/demo/beta.txt
```

Kubernetes handles the **Secret → Volume → Filesystem** wiring.