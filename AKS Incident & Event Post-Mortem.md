---
title: AKS Incident & Event Post-Mortem
date: 2026-09-16
cluster: ASP-MicroserviceApplication
node_pool: aks-nodepool1-15643098-vmss000000 (Standard_B2s_v2 - 1 Node)
tags:
  - kubernetes
  - incident-report
  - aks
  - post-mortem
---

# Cluster Timeline Analysis

| Time Ago | Component | Event Type | Description |
| :--- | :--- | :--- | :--- |
| **53m** | `mongo-deployment` | 🛑 **OOMKilled** | Initial rollout. Pod killed repeatedly (`mongod`, `mongosh`, `bash`) by node memory cgroup. |
| **53m** | `shoppingapi` | 🚀 **Deployed** | Image `hardik0811/shoppingapi:66` pulled and container started successfully. |
| **50m** | `shoppingclient` | 🚀 **Deployed** | Image `hardik0811/shoppingclient:67` scheduled, pulled, and started. |
| **43m** | `shoppingclient-lb` | 🌐 **Network** | Azure Standard Load Balancer provisioned external ingress mapping. |
| **33m** | `mongo-deployment` | 🔄 **Rollout** | ReplicaSet recreated (`74f6c4f776`), pulling a fresh `mongo` container image. |
| **6m 10s** | `mongo-deployment` | 🔄 **Rollout** | Second ReplicaSet rollout (`5c749546b4`) replaces the previous instance. |
| **5m 53s** | `shoppingapi` | 🔄 **Rollout** | API switched from `hardik0811/shoppingapi:66` to `mehmetozkaya/shoppingapi:latest`. |
| **5m 42s** | `shoppingapi-hpa` | 📈 **Autoscale** | HPA triggers initial scale from 1 to 2 replicas based on `MinReplicas`. |
| **1m 26s** | `shoppingapi-hpa` | 📈 **Autoscale Spike** | HPA triggers rapid scale-out: 2 → 4 → 8 → 10 replicas due to high CPU load. |
| **0m 41s** | `node/vmss000000` | ⚠️ **Resource Depletion** | Single `Standard_B2s_v2` node runs out of allocatable CPU (`0/1 nodes available: 1 Insufficient cpu`). |

---

# Root Cause Breakdown

### 1. The Deserialization Crash Explained
The .NET runtime crash:
```text
JsonReaderException: Unexpected character encountered while parsing value: S. Path '', line 0, position 0.