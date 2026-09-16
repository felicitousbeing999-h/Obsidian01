d Shell

Your Cloud Shell session will be ephemeral so no files or system changes will persist beyond your current session.
hardik [ ~ ]$ az acr list -o table
NAME         RESOURCE GROUP    LOCATION      SKU    LOGIN SERVER            CREATION DATE         ADMIN ENABLED
-----------  ----------------  ------------  -----  ----------------------  --------------------  ---------------
hardikarora  k8sgpt            centralindia  Basic  hardikarora.azurecr.io  2026-08-15T10:26:40Z  True
hardik [ ~ ]$ az acr show -n hardikacr --query loginServer -o tsv
The resource with name 'hardikacr' and type 'Microsoft.ContainerRegistry/registries' could not be found in subscription 'Azure for Students (94321bdf-da5f-4cc4-86eb-fe1c24335bfb)'.
hardik [ ~ ]$ az acr show -n k8sgpt --query loginServer -o tsv
The resource with name 'k8sgpt' and type 'Microsoft.ContainerRegistry/registries' could not be found in subscription 'Azure for Students (94321bdf-da5f-4cc4-86eb-fe1c24335bfb)'.
hardik [ ~ ]$ az acr show
the following arguments are required: --name/-n

Examples from command's help:
az acr show -n myregistry --query loginServer
Get the login server for an Azure Container Registry.

az acr show --name myregistry --resource-group MyResourceGroup
Get the details of an Azure Container Registry

az acr show --name myregistry --resource-group MyResourceGroup --query roleAssignmentMode
Check status of ABAC-based Repository Permission on a registry.

https://aka.ms/cli_ref
Read more about the command in reference docs
hardik [ ~ ]$ az acr show --resource-group 
k8sgpt                                              MC_k8sgpt_ASP-MicroserviceApplication_centralindia  NetworkWatcherRG
hardik [ ~ ]$ az acr show --resource-group 
k8sgpt                                              MC_k8sgpt_ASP-MicroserviceApplication_centralindia  NetworkWatcherRG
hardik [ ~ ]$ az acr show --resource-group 
k8sgpt                                              MC_k8sgpt_ASP-MicroserviceApplication_centralindia  NetworkWatcherRG
hardik [ ~ ]$ az acr show --resource-group 
argument --resource-group/-g: expected one argument

Examples from command's help:
az acr show --name myregistry --resource-group MyResourceGroup
Get the details of an Azure Container Registry

az acr show --name myregistry --resource-group MyResourceGroup --query roleAssignmentMode
Check status of ABAC-based Repository Permission on a registry.

az acr show -n myregistry --query loginServer
Get the login server for an Azure Container Registry.

https://aka.ms/cli_ref
Read more about the command in reference docs
hardik [ ~ ]$ 
hardik [ ~ ]$ 
hardik [ ~ ]$  MC_k8sgpt_ASP-MicroserviceApplication_centralindia 
bash: MC_k8sgpt_ASP-MicroserviceApplication_centralindia: command not found
hardik [ ~ ]$ az acr show --resource-group ^C
hardik [ ~ ]$ az aks list \
  --query "[].{Name:name,ResourceGroup:resourceGroup,Location:location}" \
  -o table
Name                         ResourceGroup    Location
---------------------------  ---------------  ------------
ASP-MicroserviceApplication  k8sgpt           centralindia
hardik [ ~ ]$ az devops service-endpoint list \
  --organization https://dev.azure.com/YOUR_ORG \
  --project "YOUR_PROJECT" \
  -o table
^C^C
hardik [ ~ ]$ ^C
hardik [ ~ ]$ az devops service-endpoint list   --organization https://dev.azure.com/hardikarora22cs   --project "Azure-DevOps-Full_Microservice_Pipeline"   -o table
Preview version of extension is disabled by default for extension installation, enabled for modules without stable versions. 
Please run 'az config set extension.dynamic_install_allow_preview=true or false' to config it specifically. 
The command requires the extension azure-devops. Do you want to install it now? The command will continue to run after the extension is installed. (Y/n): Y
Run 'az config set extension.use_dynamic_install=yes_without_prompt' to allow installing extensions without prompt.
The resource cannot be found.  Operation returned a 404 status code.
hardik [ ~ ]$ az extension add --name azure-devops
Extension 'azure-devops' 1.0.8 is already installed.
hardik [ ~ ]$ az devops configure \
  --defaults organization=https://dev.azure.com/hardikaroracs22 \
  project="Azure-DevOps-Full_Microservice_Pipeline"
hardik [ ~ ]$ az devops service-endpoint list -o table
Auto-detect was enabled but no Azure DevOps remote was found in the current git repository. Ensure your git remote points to an Azure DevOps URL (e.g., https://dev.azure.com/MyOrganization/...).
ID                                    Name                    Type    Is Ready    Created By
------------------------------------  ----------------------  ------  ----------  ------------
5cb756d2-0558-4b09-bd0a-16d559a10839  github.com_barbaria888  github  True        Hardik Arora
hardik [ ~ ]$ az aks get-credentials \
  --resource-group rg-devops \
  --name application
^Chardik [ ~ ]$ az aks get-credentials   --resource-group k8sgpt   --name ASP-MicroserviceApplication
Merged "ASP-MicroserviceApplication" as current context in /home/hardik/.kube/config
Converted kubeconfig to use Azure CLI authentication.
hardik [ ~ ]$ k get ns
bash: k: command not found
hardik [ ~ ]$ kubectl get ns
NAME              STATUS   AGE
application       Active   16h
default           Active   16h
kube-node-lease   Active   16h
kube-public       Active   16h
kube-system       Active   16h
hardik [ ~ ]$ az acr show \
  --name myacr \
  --resource-group k8sgpt \
  --query loginServer \
  -o tsv
(ResourceNotFound) The Resource 'Microsoft.ContainerRegistry/registries/myacr' under resource group 'k8sgpt' was not found. For more details please go to https://aka.ms/ARMResourceNotFoundFix
Code: ResourceNotFound
Message: The Resource 'Microsoft.ContainerRegistry/registries/myacr' under resource group 'k8sgpt' was not found. For more details please go to https://aka.ms/ARMResourceNotFoundFix
hardik [ ~ ]$ 