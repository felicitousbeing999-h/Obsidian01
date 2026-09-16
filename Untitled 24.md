![[Pasted image 20260913121254.png]]Yes. Since you want to **recreate the old AKS with the same name/settings**, use this one-liner:

```
az aks create -g k8sgpt -n ASP-MicroserviceApplication --location centralindia --kubernetes-version 1.35 --node-count 1 --node-vm-size Standard_B2s --attach-acr hardikarora --generate-ssh-keys
```

Then get credentials:

```
az aks get-credentials -g k8sgpt -n ASP-MicroserviceApplication --overwrite-existing
```

And verify:

```
kubectl get nodes
```

### Then create `AKS-Connection`

Once AKS is back, verify the cluster:

```
az aks show -g k8sgpt -n ASP-MicroserviceApplication -o table
```

Then we'll create the **Azure DevOps WIF service connection** named:

```
AKS-Connection
```

Your `variables.yml` can stay exactly as: