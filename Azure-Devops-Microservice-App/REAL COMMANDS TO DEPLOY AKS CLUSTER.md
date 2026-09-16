# Set variables
```BASH

ACR_NAME="hardikarora.acr.io"
RESOURCE_GROUP="k8sgpt"
CLUSTER_NAME="ASP-MicroserviceApplication"
LOCATION="centralindia"
VM_SIZE="Standard_B2s_v2"
NAMESPACE="application"
```
# 1. Create the AKS cluster with B-series compute, Entra ID, and Azure RBAC
```bash
az aks create \
  --resource-group "$RESOURCE_GROUP" \
  --name "$CLUSTER_NAME" \
  --location "$LOCATION" \
  --node-vm-size "$VM_SIZE" \
  --node-count 2 \
  --enable-aad \
  --enable-azure-rbac \
  --generate-ssh-keys
```
### 2. Assign yourself Cluster Admin (needed because of --enable-azure-rbac)


```bash
USER_ID=$(az ad signed-in-user show --query id -o tsv)
AKS_ID=$(az aks show -g "$RESOURCE_GROUP" -n "$CLUSTER_NAME" --query id -o tsv)

az role assignment create \
  --role "Azure Kubernetes Service RBAC Cluster Admin" \
  --assignee "$USER_ID" \
  --scope "$AKS_ID"
```
# 3. Get cluster user credentials
```bash
az aks get-credentials \
  --resource-group "$RESOURCE_GROUP" \
  --name "$CLUSTER_NAME" \
  --overwrite-existing
```
# 4. Create the 'application' namespace
```bash
kubectl create namespace "$NAMESPACE"
 # Verify namespace creation
kubectl get namespace "$NAMESPACE"
```


### 5. Attach ACR to AKS (so pods can pull images)

```bash
az aks update \
  --resource-group "$RESOURCE_GROUP" \
  --name "$CLUSTER_NAME" \
  --attach-acr "$ACR_NAME"
```


### 6. Quick verification commands

```bash
# Nodes
kubectl get nodes

# ACR attachment check
az aks check-acr -g "$RESOURCE_GROUP" -n "$CLUSTER_NAME" --acr "$ACR_NAME.azurecr.io"

# Role assignments on ACR (should show AcrPull for the kubelet identity)
az role assignment list \
  --scope $(az acr show -n "$ACR_NAME" --query id -o tsv) \
  --output table
```
Here’s the **exact current status** and what you should do right now.

### Current Status (from your last messages)

| Item                        | Status                          | Value                                      |
|----------------------------|----------------------------------|--------------------------------------------|
| AKS Cluster                | Created                         | `ASP-MicroserviceApplication`              |
| Resource Group             | Exists                          | `k8sgpt`                                   |
| Namespace                  | Created                         | `application`                              |
| Nodes                      | Ready (2 nodes)                 | Standard_B2s_v2                            |
| Azure RBAC                 | Enabled + Admin role assigned   | Yes                                        |
| ACR Attachment             | **Not fully done yet**          | Needs verification                         |
| Azure DevOps Service Conn  | Not confirmed                   | Need to create `AKS-Connection`            |
| Environment `dev`          | Not confirmed                   | Need to create + optional approval         |

---

### Step-by-step – What to do now

### 1. Attach ACR to AKS (Important)

```bash
az aks update \
  --resource-group k8sgpt \
  --name ASP-MicroserviceApplication \
  --attach-acr hardikarora
```

Then verify:

```bash
az aks check-acr \
  --resource-group k8sgpt \
  --name ASP-MicroserviceApplication \
  --acr hardikarora.azurecr.io
```

---

### 2. Fill `templates/common/variables.yml`

Replace the content with this:

```yaml
variables:
  - name: buildConfiguration
    value: 'Release'

  - name: dockerRegistryServiceConnection
    value: 'Azure Container Registry'          # exact name of your ACR service connection

  - name: containerRegistry
    value: 'hardikarora.azurecr.io'

  - name: imageTag
    value: '$(Build.BuildId)'

  # ===== AKS Settings =====
  - name: aksServiceConnection
    value: 'AKS-Connection'                    # exact name you will create in Azure DevOps

  - name: aksResourceGroup
    value: 'k8sgpt'

  - name: aksClusterName
    value: 'ASP-MicroserviceApplication'

  - name: aksNamespace
    value: 'application'
```

---

### 3. Create Service Connection in Azure DevOps

1. Go to **Project Settings → Service connections**
2. **New service connection** → **Kubernetes**
3. Authentication method → **Azure Subscription**
4. Select:
   - Subscription
   - Resource Group → `k8sgpt`
   - Cluster → `ASP-MicroserviceApplication`
5. Namespace → leave empty or put `application`
6. **Service connection name**: `AKS-Connection` (must match variables.yml)
7. Check **Grant access permission to all pipelines**
8. Save

---

### 4. Create Environment + Approval (Recommended)

1. Go to **Pipelines → Environments**
2. Create environment named exactly: **`dev`**
3. Click on `dev` → **Approvals and checks** → **+** → **Approvals**
4. Add yourself as Approver → Save

This will make the CD stage wait for your manual approval.

---

### 5. Branch Strategy (Current recommendation)

| Branch     | What happens                          |
|------------|---------------------------------------|
| `develop`  | Only CI runs (Build + Trivy + Push)   |
| `main`     | CI + CD runs (with manual approval)   |

This is already correctly configured in your pipelines with:

```yaml
condition: and(succeeded(), eq(variables['Build.SourceBranch'], 'refs/heads/main'))
```

No need to change anything here.

---

### Final Checklist before first full run

| Task                                      | Done? |
|-------------------------------------------|-------|
| AKS cluster running                       | ✅    |
| Namespace `application` exists            | ✅    |
| ACR attached to AKS                       | ☐     |
| `variables.yml` updated                   | ☐     |
| Service connection `AKS-Connection` created | ☐   |
| Environment `dev` created + Approval      | ☐     |
| Both pipelines have CI + CD stages        | ☐     |
| `deploy-infra.yml` template created       | ☐     |

---

Would you like me to give you the **final complete versions** of both pipeline files + the two templates (`deploy-infra.yml` + `deploy-aks.yml`) so you can just copy-paste everything?


```bash

--enable-azure-rbac \
kubectl get namespace "$NAMESPACE"CE"ce
SSH key files '/home/hardik/.ssh/id_rsa' and '/home/hardik/.ssh/id_rsa.pub' have been generated under ~/.ssh to allow SSH access to the VM. If using machines without permanent storage like Azure Cloud Shell without an attached file share, back up your keys to a safe location
{
  "aadProfile": {
    "adminGroupObjectIDs": null,
    "clientAppId": null,
    "enableAzureRbac": true,
    "managed": true,
    "serverAppId": null,
    "serverAppSecret": null,
    "tenantId": "c0177927-d841-4ce1-ae7d-c272a34a3b20"
  },
  "addonProfiles": null,
  "agentPoolProfiles": [
    {
      "artifactStreamingProfile": null,
      "availabilityZones": null,
      "capacityReservationGroupId": null,
      "count": 2,
      "creationData": null,
      "currentOrchestratorVersion": "1.35.7",
      "eTag": "7c3411a1-e6d4-4774-8801-58d5c2cc720f",
      "enableAutoScaling": false,
      "enableEncryptionAtHost": false,
      "enableFips": false,
      "enableNodePublicIp": false,
      "enableUltraSsd": false,
      "gatewayProfile": null,
      "gpuInstanceProfile": null,
      "gpuProfile": null,
      "hostGroupId": null,
      "kubeletConfig": null,
      "kubeletDiskType": "OS",
      "linuxOsConfig": null,
      "localDnsProfile": null,
      "maxCount": null,
      "maxPods": 250,
      "messageOfTheDay": null,
      "minCount": null,
      "mode": "System",
      "name": "nodepool1",
      "networkProfile": null,
      "nodeImageVersion": "AKSUbuntu-2404gen2containerd-202608.26.0",
      "nodeLabels": null,
      "nodePublicIpPrefixId": null,
      "nodeTaints": null,
      "orchestratorVersion": "1.35",
      "osDiskSizeGb": 128,
      "osDiskType": "Managed",
      "osSku": "Ubuntu",
      "osType": "Linux",
      "podIpAllocationMode": null,
      "podSubnetId": null,
      "powerState": {
        "code": "Running"
      },
      "provisioningState": "Succeeded",
      "proximityPlacementGroupId": null,
      "scaleDownMode": "Delete",
      "scaleSetEvictionPolicy": null,
      "scaleSetPriority": null,
      "securityProfile": {
        "enableSecureBoot": false,
        "enableVtpm": false,
        "sshAccess": "LocalUser"
      },
      "spotMaxPrice": null,
      "status": null,
      "tags": null,
      "type": "VirtualMachineScaleSets",
      "upgradeSettings": {
        "drainTimeoutInMinutes": null,
        "maxSurge": "10%",
        "maxUnavailable": "0",
        "nodeSoakDurationInMinutes": null,
        "undrainableNodeBehavior": null
      },
      "virtualMachineNodesStatus": null,
      "virtualMachinesProfile": null,
      "vmSize": "Standard_B2s_v2",
      "vnetSubnetId": null,
      "windowsProfile": null,
      "workloadRuntime": null
    }
  ],
  "aiToolchainOperatorProfile": null,
  "apiServerAccessProfile": null,
  "autoScalerProfile": null,
  "autoUpgradeProfile": {
    "nodeOsUpgradeChannel": "NodeImage",
    "upgradeChannel": null
  },
  "azureMonitorProfile": null,
  "azurePortalFqdn": "asp-micros-k8sgpt-94321b-uqwfousp.portal.hcp.centralindia.azmk8s.io",
  "bootstrapProfile": {
    "artifactSource": "Direct",
    "containerRegistryId": null
  },
  "currentKubernetesVersion": "1.35.7",
  "disableLocalAccounts": false,
  "diskEncryptionSetId": null,
  "dnsPrefix": "ASP-Micros-k8sgpt-94321b",
  "eTag": "c488606b-1b5c-48f2-9668-03c982a0dc31",
  "enableRbac": true,
  "extendedLocation": null,
  "fqdn": "asp-micros-k8sgpt-94321b-uqwfousp.hcp.centralindia.azmk8s.io",
  "fqdnSubdomain": null,
  "hostedSystemProfile": {
    "enabled": false,
    "nodeSubnetId": null,
    "systemNodeSubnetId": null
  },
  "httpProxyConfig": null,
  "id": "/subscriptions/94321bdf-da5f-4cc4-86eb-fe1c24335bfb/resourcegroups/k8sgpt/providers/Microsoft.ContainerService/managedClusters/ASP-MicroserviceApplication",
  "identity": {
    "delegatedResources": null,
    "principalId": "84810c44-f851-4138-98c9-68907ede462f",
    "tenantId": "c0177927-d841-4ce1-ae7d-c272a34a3b20",
    "type": "SystemAssigned",
    "userAssignedIdentities": null
  },
  "identityProfile": {
    "kubeletidentity": {
      "clientId": "d46bf8d0-1532-4544-b297-0fdf14abc03a",
      "objectId": "62543f7f-9302-4bf6-8dfd-4b9b6edad49b",
      "resourceId": "/subscriptions/94321bdf-da5f-4cc4-86eb-fe1c24335bfb/resourcegroups/MC_k8sgpt_ASP-MicroserviceApplication_centralindia/providers/Microsoft.ManagedIdentity/userAssignedIdentities/ASP-MicroserviceApplication-agentpool"
    }
  },
  "ingressProfile": null,
  "kind": "Base",
  "kubernetesVersion": "1.35",
  "linuxProfile": {
    "adminUsername": "azureuser",
    "ssh": {
      "publicKeys": [
        {
          "keyData": "ssh-rsa AAAAB3NzaC1yc2EAAAADAQABAAABAQCeFtcTO5Hy22boh8/psnXW/9E1HWy93aXDYYM1dRlvHSv5ue32eUlJ+zDLMdjJTKtQJoUwramFpuuTM9+xf1rLCGc2hGgiku8AVQ/hl+i6PvTYX9GJjAzXtIVICLdlWuVVpOY87K4w6ii954RD2ibe5TUvJ7qiYE41Ttxwqd5/KFLkAtyiU5KVD2G01eFuzq5UOhyeC6TPM2QxZpnPxn0W/Wnb2bDm/jg+spyk31JL44Glz98HmZhoPXZ0NwFbuCaXirKCN79wV1FRIRVMGvJ6U16UHgbwOvOo7qUr/XFGqDH9BqzftBAIseXJ9e71Rbmt4bo+G30jm70cMetzLpVf"
        }
      ]
    }
  },
  "location": "centralindia",
  "maxAgentPools": 100,
  "metricsProfile": {
    "costAnalysis": {
      "enabled": false
    }
  },
  "name": "ASP-MicroserviceApplication",
  "networkProfile": {
    "advancedNetworking": null,
    "dnsServiceIp": "10.0.0.10",
    "ipFamilies": [
      "IPv4"
    ],
    "loadBalancerProfile": {
      "allocatedOutboundPorts": null,
      "backendPoolType": "nodeIPConfiguration",
      "effectiveOutboundIPs": [
        {
          "id": "/subscriptions/94321bdf-da5f-4cc4-86eb-fe1c24335bfb/resourceGroups/MC_k8sgpt_ASP-MicroserviceApplication_centralindia/providers/Microsoft.Network/publicIPAddresses/37125586-a31b-4146-8448-a9cab1bc2def",
          "resourceGroup": "MC_k8sgpt_ASP-MicroserviceApplication_centralindia"
        }
      ],
      "enableMultipleStandardLoadBalancers": null,
      "idleTimeoutInMinutes": null,
      "managedOutboundIPs": {
        "count": 1,
        "countIpv6": null
      },
      "outboundIPs": null,
      "outboundIpPrefixes": null
    },
    "loadBalancerSku": "standard",
    "natGatewayProfile": null,
    "networkDataplane": "azure",
    "networkMode": null,
    "networkPlugin": "azure",
    "networkPluginMode": "overlay",
    "networkPolicy": "none",
    "outboundType": "loadBalancer",
    "podCidr": "10.244.0.0/16",
    "podCidrs": [
      "10.244.0.0/16"
    ],
    "serviceCidr": "10.0.0.0/16",
    "serviceCidrs": [
      "10.0.0.0/16"
    ],
    "staticEgressGatewayProfile": null
  },
  "nodeProvisioningProfile": {
    "defaultNodePools": null,
    "mode": "Manual"
  },
  "nodeResourceGroup": "MC_k8sgpt_ASP-MicroserviceApplication_centralindia",
  "nodeResourceGroupProfile": null,
  "oidcIssuerProfile": {
    "enabled": true,
    "issuerUrl": "https://centralindia.oic.prod-aks.azure.com/c0177927-d841-4ce1-ae7d-c272a34a3b20/0a338a64-d973-42d1-bd47-b9dc3dfce9d2/"
  },
  "podIdentityProfile": null,
  "powerState": {
    "code": "Running"
  },
  "privateFqdn": null,
  "privateLinkResources": null,
  "provisioningState": "Succeeded",
  "publicNetworkAccess": null,
  "resourceGroup": "k8sgpt",
  "resourceUid": "6aa552b925b920000189853e",
  "schedulerProfile": null,
  "securityProfile": {
    "azureKeyVaultKms": null,
    "customCaTrustCertificates": null,
    "defender": null,
    "imageCleaner": null,
    "workloadIdentity": null
  },
  "serviceMeshProfile": null,
  "servicePrincipalProfile": {
    "clientId": "msi",
    "secret": null
  },
  "sku": {
    "name": "Base",
    "tier": "Free"
  },
  "status": null,
  "storageProfile": {
    "blobCsiDriver": null,
    "diskCsiDriver": {
      "enabled": true
    },
    "fileCsiDriver": {
      "enabled": true
    },
    "snapshotController": {
      "enabled": true
    }
  },
  "supportPlan": "KubernetesOfficial",
  "systemData": null,
  "tags": null,
  "type": "Microsoft.ContainerService/ManagedClusters",
  "upgradeSettings": null,
  "windowsProfile": null,
  "workloadAutoScalerProfile": {
    "keda": null,
    "verticalPodAutoscaler": null
  }
}
Merged "ASP-MicroserviceApplication" as current context in /home/hardik/.kube/config
Converted kubeconfig to use Azure CLI authentication.
Error from server (Forbidden): namespaces is forbidden: User "4bb25e27-f053-409c-85cb-fb3b4072d37b" cannot create resource "namespaces" in API group "" at the cluster scope: User does not have access to the resource in Azure. Update role assignment to allow access.
Error from server (Forbidden): namespaces "application" is forbidden: User "4bb25e27-f053-409c-85cb-fb3b4072d37b" cannot get resource "namespaces" in API group "" in the namespace "application": User does not have access to the resource in Azure. Update role assignment to allow access.
hardik [ ~ ]$ # Get your user principal object ID and the AKS cluster resource ID
USER_ID=$(az ad signed-in-user show --query id -o tsv)
AKS_ID=$(az aks show --resource-group k8sgpt --name ASP-MicroserviceApplication --query id -o tsv)

# Assign the RBAC Cluster Admin role
az role assignment create \
  --role "Azure Kubernetes Service RBAC Cluster Admin" \
  --assignee "$USER_ID" \
  --scope "$AKS_ID"
{
  "condition": null,
  "conditionVersion": null,
  "createdBy": null,
  "createdOn": "2026-09-12T13:34:19.462744+00:00",
  "delegatedManagedIdentityResourceId": null,
  "description": null,
  "id": "/subscriptions/94321bdf-da5f-4cc4-86eb-fe1c24335bfb/resourcegroups/k8sgpt/providers/Microsoft.ContainerService/managedClusters/ASP-MicroserviceApplication/providers/Microsoft.Authorization/roleAssignments/8fee10f6-56ef-4207-8aee-07df2045dd55",
  "name": "8fee10f6-56ef-4207-8aee-07df2045dd55",
  "principalId": "4bb25e27-f053-409c-85cb-fb3b4072d37b",
  "principalType": "User",
  "resourceGroup": "k8sgpt",
  "roleDefinitionId": "/subscriptions/94321bdf-da5f-4cc4-86eb-fe1c24335bfb/providers/Microsoft.Authorization/roleDefinitions/b1ff04bb-8a4e-4dc4-8eb5-8693973ce19b",
  "scope": "/subscriptions/94321bdf-da5f-4cc4-86eb-fe1c24335bfb/resourcegroups/k8sgpt/providers/Microsoft.ContainerService/managedClusters/ASP-MicroserviceApplication",
  "systemData": null,
  "type": "Microsoft.Authorization/roleAssignments",
  "updatedBy": "4bb25e27-f053-409c-85cb-fb3b4072d37b",
  "updatedOn": "2026-09-12T13:34:19.749748+00:00"
}
hardik [ ~ ]$ az role assignment list --assignee "$USER_ID" --scope "$AKS_ID" --query "[].roleDefinitionName" -o tsv
Azure Kubernetes Service RBAC Cluster Admin
hardik [ ~ ]$ az aks get-credentials --resource-group k8sgpt --name ASP-MicroserviceApplication --overwrite-existing
Merged "ASP-MicroserviceApplication" as current context in /home/hardik/.kube/config
Converted kubeconfig to use Azure CLI authentication.
hardik [ ~ ]$ kubectl get nodes
NAME                                STATUS   ROLES    AGE     VERSION
aks-nodepool1-39813185-vmss000000   Ready    <none>   7m4s    v1.35.7
aks-nodepool1-39813185-vmss000001   Ready    <none>   6m51s   v1.35.7
hardik [ ~ ]$ kubectl create namespace application
kubectl get namespace application
namespace/application created
NAME          STATUS   AGE
application   Active   2s
hardik [ ~ ]$ # 1. Fetch your Container Registry login server (if an
















```


``` bash
# Set variables
RESOURCE_GROUP="k8sgpt"
CLUSTER_NAME="ASP-MicroserviceApplication"
LOCATION="centralindia"
VM_SIZE="Standard_B2s_v2"
NAMESPACE="application"

# 1. Create the AKS cluster with B-series compute, Entra ID, and Azure RBAC
az aks create \
  --resource-group "$RESOURCE_GROUP" \
  --name "$CLUSTER_NAME" \
  --location "$LOCATION" \
  --node-vm-size "$VM_SIZE" \
  --node-count 2 \
  --enable-aad \
  --enable-azure-rbac \
  --generate-ssh-keys

# 2. Get cluster user credentials
az aks get-credentials \
  --resource-group "$RESOURCE_GROUP" \
  --name "$CLUSTER_NAME" \
  --overwrite-existing

# 3. Create the 'application' namespace
kubectl create namespace "$NAMESPACE"

# Verify namespace creationACR_NAME=""
kubectl get namespace "$NAMESPACE"
```
