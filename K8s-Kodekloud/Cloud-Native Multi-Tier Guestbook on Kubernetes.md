![Pasted image 20260903191228.png](https://github.com/felicitousbeing999-h/Obsidian01/blob/main/Pasted%20image%2020260903191228.png)




> `A resilient, scalable multi-tier web application deployed on Kubernetes featuring an asynchronous Redis read/write replica cluster, a stateless PHP frontend tier with self-healing capabilities, and explicit resource governance.`


## Architecture Overview

The system isolates read and write operations across the data tier to optimize throughput and fault tolerance:
* **Ingress / Edge**: NodePort Service exposing the frontend on port `30009`.
* **Presentation Tier**: Stateless, load-balanced PHP web servers scaled to 3 replicas.
* **Service Discovery**: Decoupled service addressing via CoreDNS (`GET_HOSTS_FROM=dns`).
* **Cache / Storage Tier**: Single-primary write node (`redis-master`) asynchronously replicating state changes to 2 read replicas (`redis-slave`).

```mermaid
graph TD
    Client([Client / Browser]) -->|HTTP :30009| SvcNodePort[Service: frontend<br/>NodePort :80 / :30009]

    subgraph FrontendTier ["Frontend Tier (Replicas: 3, Requests: 100m CPU / 100Mi RAM)"]
        SvcNodePort --> FE1[Pod: frontend-1]
        SvcNodePort --> FE2[Pod: frontend-2]
        SvcNodePort --> FE3[Pod: frontend-3]
    end

    subgraph ServiceBus ["CoreDNS Discovery Layer"]
        FE1 -.->|Writes :6379| SvcMaster[Service: redis-master<br/>ClusterIP :6379]
        FE2 -.->|Writes :6379| SvcMaster
        FE3 -.->|Writes :6379| SvcMaster

        FE1 -.->|Reads :6379| SvcSlave[Service: redis-slave / redis-follower<br/>ClusterIP :6379]
        FE2 -.->|Reads :6379| SvcSlave
        FE3 -.->|Reads :6379| SvcSlave
    end

    subgraph StorageTier ["Data Tier (Redis Master-Replica)"]
        SvcMaster --> RM[Pod: redis-master<br/>Replicas: 1]
        SvcSlave --> RS1[Pod: redis-slave-1<br/>Replicas: 2]
        SvcSlave --> RS2[Pod: redis-slave-2]
        RM ==>|Replication :6379| RS1
        RM ==>|Replication :6379| RS2
    end

    classDef svc fill:#1e293b,stroke:#3b82f6,stroke-width:2px,color:#fff;
    classDef pod fill:#0f172a,stroke:#64748b,stroke-width:1.5px,color:#fff;
    class SvcNodePort,SvcMaster,SvcSlave svc;
    class FE1,FE2,FE3,RM,RS1,RS2 pod;
````

## Workload Specifications

|**Component**|**Kind**|**Target Port**|**Endpoints**|**CPU Request**|**Memory Request**|**Discovery Anchor**|
|---|---|---|---|---|---|---|
|`redis-master`|Deployment / ClusterIP|`6379/TCP`|1 Pod|`100m`|`100Mi`|`redis-master:6379`|
|`redis-slave`|Deployment / ClusterIP|`6379/TCP`|2 Pods|`100m`|`100Mi`|`redis-slave:6379`|
|`redis-follower`|Service (ClusterIP)|`6379/TCP`|Target: `redis-slave`|N/A|N/A|`redis-follower:6379`|
|`frontend`|Deployment / NodePort|`80:30009`|3 Pods|`100m`|`100Mi`|Node IP : `30009`|

## Production Manifest (`manifests.yaml`)

YAML

```
# ------------------------------------------------------------------------------
# 1. REDIS MASTER (Single Write Authority)
# ------------------------------------------------------------------------------
apiVersion: apps/v1
kind: Deployment
metadata:
  name: redis-master
  labels:
    app: redis
    role: master
spec:
  replicas: 1
  selector:
    matchLabels:
      app: redis
      role: master
  template:
    metadata:
      labels:
        app: redis
        role: master
    spec:
      containers:
        - name: master-redis-datacenter
          image: redis
          ports:
            - containerPort: 6379
          resources:
            requests:
              cpu: 100m
              memory: 100Mi
---
apiVersion: v1
kind: Service
metadata:
  name: redis-master
  labels:
    app: redis
    role: master
spec:
  type: ClusterIP
  selector:
    app: redis
    role: master
  ports:
    - port: 6379
      targetPort: 6379
---
# ------------------------------------------------------------------------------
# 2. REDIS SLAVES (Horizontally Scaled Read Pool)
# ------------------------------------------------------------------------------
apiVersion: apps/v1
kind: Deployment
metadata:
  name: redis-slave
  labels:
    app: redis-slave
spec:
  replicas: 2
  selector:
    matchLabels:
      app: redis-slave
  template:
    metadata:
      labels:
        app: redis-slave
    spec:
      containers:
        - name: slave-redis-datacenter
          image: gcr.io/google_samples/gb-redisslave:v3
          ports:
            - containerPort: 6379
          env:
            - name: GET_HOSTS_FROM
              value: dns
          resources:
            requests:
              cpu: 100m
              memory: 100Mi
---
apiVersion: v1
kind: Service
metadata:
  name: redis-slave
  labels:
    app: redis-slave
spec:
  type: ClusterIP
  selector:
    app: redis-slave
  ports:
    - port: 6379
      targetPort: 6379
---
apiVersion: v1
kind: Service
metadata:
  name: redis-follower
  labels:
    app: redis-slave
spec:
  type: ClusterIP
  selector:
    app: redis-slave
  ports:
    - port: 6379
      targetPort: 6379
---
# ------------------------------------------------------------------------------
# 3. WEB FRONTEND TIER (Stateless Presentation Engine)
# ------------------------------------------------------------------------------
apiVersion: apps/v1
kind: Deployment
metadata:
  name: frontend
  labels:
    app: frontend
spec:
  replicas: 3
  selector:
    matchLabels:
      app: frontend
  template:
    metadata:
      labels:
        app: frontend
    spec:
      containers:
        - name: php-redis-datacenter
          image: gcr.io/google-samples/gb-frontend@sha256:a908df8486ff66f2c4daa0d3d8a2fa09846a1fc8efd65649c0109695c7c5cbff
          ports:
            - containerPort: 80
          env:
            - name: GET_HOSTS_FROM
              value: dns
          resources:
            requests:
              cpu: 100m
              memory: 100Mi
---
apiVersion: v1
kind: Service
metadata:
  name: frontend
  labels:
    app: frontend
spec:
  type: NodePort
  selector:
    app: frontend
  ports:
    - port: 80
      targetPort: 80
      nodePort: 30009
```

## Deployment & Verification Runbook

### 1. Apply Deployment Manifests

Bash

```
kubectl apply -f manifests.yaml
```

### 2. Verify Pod Scheduling and Resource Allocations




Ensure all 6 pods reach the `Running` state without pending on scheduling limits:
![Pasted image 20260903192705.png](https://github.com/felicitousbeing999-h/Obsidian01/blob/main/Pasted%20image%2020260903192705.png)
  

### 3. Validate Service Endpoints and Network Routing

Confirm target endpoints match the running container IP addresses:

  

Bash

```
kubectl get endpoints redis-master redis-slave redis-follower frontend
```

### 4. Verify Internal ClusterDNS Resolution

Run an ad-hoc debugging container to verify DNS lookups across cluster namespaces:

  

Bash

```
kubectl run net-tool --rm -it --restart=Never --image=busybox:1.36 -- nslookup redis-master
```
![Pasted image 20260903191807.png](https://github.com/felicitousbeing999-h/Obsidian01/blob/main/Pasted%20image%2020260903191807.png)
### 5. Ingress Validation

Test external connectivity through the allocated NodePort (`30009`):

  ![Pasted image 20260903191839.png](https://github.com/felicitousbeing999-h/Obsidian01/blob/main/Pasted%20image%2020260903191839.png)

Bash

```
NODE_IP=$(kubectl get nodes -o jsonpath='{.items[0].status.addresses[?(@.type=="InternalIP")].address}')
curl -I http://${NODE_IP}:30009
```

A valid `HTTP/1.1 200 OK` response confirms complete end-to-end routing.