	
	
> 	Date: 2026-09-13  
> 	Project: `Azure-DevOps-Full_Microservice_Pipeline`  
> 	Goal: Get .NET microservice CI pipelines building, scanning, and pushing correct images to Azure Container Registry.
	
	---
	
	## 1. Today's Objective
	
	Build a working CI flow:
	
	```mermaid
	flowchart LR
	    G[GitHub] --> A[Azure DevOps]
	    A --> R[.NET Restore]
	    R --> B[.NET Build]
	    B --> D[Docker Build]
	    D --> T[Trivy Scan]
	    T --> P[Push to ACR]
	```
	
	Current scope:
	
	```text
	CI only
	.NET 8
	Docker
	Trivy
	ACR
	```
	
	Not currently working on:
	
	```text
	AKS
	CD
	Policy
	Governance
	```
	
	AKS was intentionally deleted to avoid unnecessary Azure cost.
	
	---
	
	# 2. Azure Environment
	
	## Azure DevOps
	
	Organization:
	
	```text
	https://dev.azure.com/hardikaroracs22
	```
	
	Project:
	
	```text
	Azure-DevOps-Full_Microservice_Pipeline
	```
	
	## Azure Subscription
	
	```text
	Azure for Students
	```
	
	## ACR
	
	```text
	Name: hardikarora
	Login Server: hardikarora.azurecr.io
	Region: Central India
	```
	
	Target repositories:
	
	```text
	hardikarora.azurecr.io/shoppingapi
	hardikarora.azurecr.io/shoppingclient
	```
	
	---
	
	# 3. Service Connections
	
	Current Azure DevOps service connections:
	
	```text
	Azure Container Registry
	github.com_barbaria888
	```
	
	Important:
	
	```yaml
	containerRegistry: 'Azure Container Registry'
	```
	
	### Do NOT use
	
	```yaml
	containerRegistry: 'ACR-Connection'
	```
	
	or:
	
	```yaml
	containerRegistry: '$(dockerRegistryServiceConnection)'
	```
	
	The latter previously caused Azure DevOps to interpret the variable literally as a service connection name.
	
	### Working approach
	
	Use the actual service connection name directly:
	
	```yaml
	containerRegistry: 'Azure Container Registry'
	```
	
	---
	
	# 4. .NET Migration
	
	The project was migrated from old .NET versions to:
	
	```text
	.NET 8
	```
	
	Common pipeline SDK:
	
	```yaml
	- task: UseDotNet@2
	  inputs:
	    packageType: 'sdk'
	    version: '8.0.x'
	```
	
	Previously this failed when using:
	
	```yaml
	version: '$(dotnetVersion)'
	```
	
	Azure DevOps reported:
	
	```text
	Version $(dotnetVersion) is not allowed
	```
	
	### Current decision
	
	Keep the SDK version explicit:
	
	```yaml
	version: '8.0.x'
	```
	
	---
	
	# 5. Actual Repository Structure
	
	Important paths discovered during debugging:
	
	```text
	Shopping/
	├── Shopping.API/
	│   ├── Shopping.API.csproj
	│   └── Dockerfile
	│
	└── Shopping.Client/
	    ├── Shopping.Client.csproj
	    └── Dockerfile
	```
	
	Actual API project:
	
	```text
	Shopping/Shopping.API/Shopping.API.csproj
	```
	
	Actual Client project:
	
	```text
	Shopping/Shopping.Client/Shopping.Client.csproj
	```
	
	---
	
	# 6. Dockerfile Problem
	
	The API Dockerfile contains:
	
	```dockerfile
	COPY ["Shopping.API/Shopping.API.csproj", "Shopping.API/"]
	```
	
	Therefore the Docker build context **must be**:
	
	```text
	Shopping/
	```
	
	NOT:
	
	```text
	repository root
	```
	
	### Previous failure
	
	```text
	COPY ["Shopping.API/Shopping.API.csproj", "Shopping.API/"]
	not found
	```
	
	### Root cause
	
	Docker was building from the repository root.
	
	### Fix
	
	```yaml
	buildContext: '$(Build.SourcesDirectory)/Shopping'
	```
	
	This allows the Dockerfile to resolve:
	
	```text
	Shopping.API/Shopping.API.csproj
	```
	
	correctly.
	
	### Important
	
	The API Dockerfile was intentionally **not changed**.
	
	The pipeline was fixed instead.
	
	---
	
	# 7. Generic Build Template
	
	Current:
	
	```text
	templates/ci/build-dotnet.yml
	```
	
	Purpose:
	
	```text
	Restore
	Build
	Test
	Docker Build
	```
	
	Important parameters:
	
	```yaml
	parameters:
	  - name: projectPath
	    type: string
	
	  - name: dockerfilePath
	    type: string
	
	  - name: imageRepository
	    type: string
	
	  - name: runTests
	    type: boolean
	    default: true
	```
	
	Docker build:
	
	```yaml
	- task: Docker@2
	  displayName: 'Build Docker image'
	  inputs:
	    containerRegistry: 'Azure Container Registry'
	    repository: ${{ parameters.imageRepository }}
	    command: 'build'
	    Dockerfile: ${{ parameters.dockerfilePath }}
	    buildContext: '$(Build.SourcesDirectory)/Shopping'
	    tags: |
	      $(imageTag)
	      latest
	    arguments: '--build-arg BUILD_CONFIGURATION=$(buildConfiguration)'
	```
	
	The template should remain generic.
	
	Do NOT hard-code:
	
	```text
	shoppingapi
	```
	
	inside the generic template.
	
	---
	
	# 8. Major Bug Found Today: API vs Client Repository Mismatch
	
	This was one of the most important debugging issues.
	
	A pipeline could successfully build:
	
	```text
	shoppingapi
	```
	
	but then attempt to push:
	
	```text
	shoppingclient
	```
	
	Example:
	
	```text
	Build:
	***/shoppingapi:31
	***/shoppingapi:latest
	
	Push:
	***/shoppingclient:31
	```
	
	Docker then reported:
	
	```text
	An image does not exist locally with the tag:
	***/shoppingclient
	```
	
	## Root cause
	
	The pipeline had inconsistent repository names.
	
	Example of the broken configuration:
	
	```yaml
	imageRepository: 'shoppingapi'
	```
	
	for build and Trivy, but:
	
	```yaml
	repository: 'shoppingclient'
	```
	
	for push.
	
	Therefore:
	
	```text
	Build → API
	Scan  → API
	Push  → Client
	```
	
	This is invalid.
	
	---
	
	# 9. Rule Discovered
	
	For each microservice, the following three things must always match:
	
	```text
	imageRepository
	        ↓
	Docker Build
	        ↓
	Trivy Scan
	        ↓
	ACR Push
	```
	
	For API:
	
	```text
	shoppingapi
	```
	
	For Client:
	
	```text
	shoppingclient
	```
	
	Never mix them.
	
	---
	
	# 10. Correct API Pipeline Identity
	
	```yaml
	imageRepository: 'shoppingapi'
	
	projectPath: 'Shopping/Shopping.API/Shopping.API.csproj'
	
	dockerfilePath: 'Shopping/Shopping.API/Dockerfile'
	```
	
	Trigger:
	
	```yaml
	paths:
	  include:
	    - Shopping/Shopping.API/**
	    - pipelines/**
	```
	
	Build:
	
	```yaml
	imageRepository: $(imageRepository)
	```
	
	Trivy:
	
	```yaml
	imageRepository: $(imageRepository)
	```
	
	Push:
	
	```yaml
	repository: $(imageRepository)
	```
	
	Result:
	
	```text
	hardikarora.azurecr.io/shoppingapi:<tag>
	hardikarora.azurecr.io/shoppingapi:latest
	```
	
	---
	
	# 11. Correct Client Pipeline Identity
	
	The client pipeline was found to still contain API values.
	
	Broken:
	
	```yaml
	parameters:
	  projectPath: 'Shopping/Shopping.API/Shopping.API.csproj'
	  dockerfilePath: 'Shopping/Shopping.API/Dockerfile'
	  imageRepository: 'shoppingapi'
	```
	
	and:
	
	```yaml
	imageRepository: 'shoppingapi'
	```
	
	while the variables and push were using:
	
	```text
	shoppingclient
	```
	
	This caused the same mismatch.
	
	## Correct Client configuration
	
	```yaml
	imageRepository: 'shoppingclient'
	
	projectPath: 'Shopping/Shopping.Client/Shopping.Client.csproj'
	
	dockerfilePath: 'Shopping/Shopping.Client/Dockerfile'
	```
	
	Trigger:
	
	```yaml
	paths:
	  include:
	    - Shopping/Shopping.Client/**
	    - pipelines/**
	```
	
	Template:
	
	```yaml
	- template: templates/ci/build-dotnet.yml
	  parameters:
	    projectPath: $(projectPath)
	    dockerfilePath: $(dockerfilePath)
	    imageRepository: $(imageRepository)
	    runTests: false
	```
	
	Trivy:
	
	```yaml
	- template: templates/security/trivy-scan.yml
	  parameters:
	    imageRepository: $(imageRepository)
	    failOnSeverity: 'HIGH,CRITICAL'
	```
	
	Push:![Pasted image 20260913132421.png](https://github.com/felicitousbeing999-h/Obsidian01/blob/main/Pasted%20image%2020260913132421.png)
	
	```yaml
	- task: Docker@2
	  displayName: 'Push Shopping Client image to ACR'
	  inputs:
	    containerRegistry: 'Azure Container Registry'
	    repository: $(imageRepository)
	    command: 'push'
	    tags: |
	      $(imageTag)
	      latest
	```
	
	Expected result:
	
	```text
	hardikarora.azurecr.io/shoppingclient:<tag>
	hardikarora.azurecr.io/shoppingclient:latest
	```
	
	---
	
	# 12. Trivy Debugging
	
	Trivy was successfully installed and downloaded its vulnerability database.
	
	It detected:
	
	```text
	Debian 12.15
	92 packages
	.NET language files
	```
	
	The scan found HIGH/CRITICAL vulnerabilities.
	
	Because the pipeline was configured to fail on those findings, Bash returned:
	
	```text
	exit code 1
	```
	
	Therefore:
	
	```text
	Trivy failed
	↓
	Pipeline stopped
	↓
	ACR push never happened
	```
	
	This was expected behavior from the security gate.
	
	---
	
	# 13. Temporary Trivy Strategy
	
	For now, the goal is to verify the entire CI path including ACR push.
	
	Therefore Trivy should temporarily report vulnerabilities without blocking.
	
	Use:
	
	```bash
	trivy image \
	  --exit-code 0 \
	  --severity HIGH,CRITICAL \
	  --format table \
	  --output trivy-report.txt \
	  $(containerRegistry)/${{ parameters.imageRepository }}:$(imageTag)
	```
	
	Meaning:
	
	```text
	Trivy still scans
	Trivy still reports
	Trivy still publishes report
	Trivy does NOT stop pipeline
	```
	
	Later:
	
	```text
	--exit-code 0
	```
	
	can become:
	
	```text
	--exit-code 1
	```
	
	when the security gate is intentionally enforced.
	
	---
	
	# 14. Current Trivy Template
	
	```yaml
	parameters:
	  - name: imageRepository
	    type: string
	
	  - name: failOnSeverity
	    type: string
	    default: 'HIGH,CRITICAL'
	
	steps:
	
	  - script: |
	      curl -sfL https://raw.githubusercontent.com/aquasecurity/trivy/main/contrib/install.sh | sh -s -- -b /usr/local/bin
	
	      trivy image \
	        --exit-code 0 \
	        --severity ${{ parameters.failOnSeverity }} \
	        --format table \
	        --output trivy-report.txt \
	        $(containerRegistry)/${{ parameters.imageRepository }}:$(imageTag)
	
	    displayName: 'Trivy Container Scan'
	
	  - task: PublishPipelineArtifact@1
	    displayName: 'Publish Trivy Report'
	    inputs:
	      targetPath: 'trivy-report.txt'
	      artifact: 'trivy-${{ parameters.imageRepository }}'
	    condition: always()
	```
	
	---
	
	# 15. Pipeline Variables
	
	Current:
	
	```yaml
	variables:
	  - name: dotnetVersion
	    value: '8.0.x'
	
	  - name: buildConfiguration
	    value: 'Release'
	
	  - name: dockerRegistryServiceConnection
	    value: 'Azure Container Registry'
	
	  - name: containerRegistry
	    value: 'hardikarora.azurecr.io'
	
	  - name: aksServiceConnection
	    value: 'AKS-Connection'
	
	  - name: aksNamespace
	    value: 'application'
	
	  - name: imageTag
	    value: '$(Build.BuildId)'
	```
	
	Important:
	
	```text
	imageTag = Build.BuildId
	```
	
	So a build might produce:
	
	```text
	shoppingapi:31
	shoppingapi:latest
	```
	
	---
	
	# 16. YAML Problems Encountered
	
	One pipeline/template failure pointed to:
	
	```text
	/pipelines/templates/ci/build-dotnet.yml
	Line 1, Col 1
	```
	
	The likely issue was Markdown formatting accidentally being included in the YAML file.
	
	YAML files must start directly with YAML.
	
	Correct:
	
	```yaml
	parameters:
	```
	
	NOT:
	
	````text
	```yaml
	parameters:
	```
	````
	
	Obsidian Markdown fences belong in documentation, not inside the actual `.yml` file.
	
	---
	
	# 17. AKS Cost Decision
	
	AKS was deleted today to avoid unnecessary Azure cost.
	
	Command used:
	
	```bash
	az aks delete \
	  --name ASP-MicroserviceApplication \
	  --resource-group k8sgpt \
	  --yes \
	  --no-wait
	```
	
	Therefore the current project is intentionally:
	
	```mermaid
	flowchart LR
	    G[GitHub] --> A[Azure DevOps]
	    A --> N[.NET 8]
	    N --> D[Docker Build]
	    D --> T[Trivy]
	    T --> R[Azure Container Registry]
	```
	
	No AKS dependency right now.
	
	---
	
	# 18. Current Architecture
	
	```mermaid
	flowchart LR
	    G[GitHub] --> P[Pull Request / Push]
	
	    P --> A[Azure DevOps]
	
	    A --> R[.NET Restore]
	    R --> B[.NET Build]
	    B --> D[Docker Build]
	
	    D --> T[Trivy Container Scan]
	
	    T --> ACR[Azure Container Registry]
	
	    ACR --> API[shoppingapi]
	    ACR --> CLIENT[shoppingclient]
	```
	
	Repositories:
	
	```text
	hardikarora.azurecr.io/
	├── shoppingapi
	└── shoppingclient
	```
	
	---
	
	# 19. DevSecOps Architecture Direction
	
	The project intentionally excludes:
	
	```text
	Policy
	Governance
	```
	
	The security architecture is focused on actual pipeline security:
	
	```mermaid
	flowchart TD
	    G[GitHub] --> PR[Pull Request]
	
	    PR --> CS[Code Security]
	    PR --> IS[IaC Security]
	    PR --> SS[Secrets]
	
	    CS --> CODEQL[CodeQL]
	    IS --> CHECKOV[Checkov]
	    SS --> SECRET[Secret Scanning]
	
	    CODEQL --> BUILD[Build / Test]
	    CHECKOV --> BUILD
	    SECRET --> BUILD
	
	    BUILD --> TRIVY[Trivy]
	    TRIVY --> ACR[Azure Container Registry]
	```
	
	This keeps the portfolio focused on:
	
	```text
	DevSecOps
	CI/CD
	Container Security
	IaC Security
	Cloud
	```
	
	rather than Azure governance features.
	
	---
	
	# 20. What Was Actually Achieved Today
	
	## Fixed
	
	-  Confirmed actual Azure DevOps project
	    
	-  Confirmed ACR service connection
	    
	-  Fixed invalid `ACR-Connection` reference
	    
	-  Fixed service connection variable resolution issue
	    
	-  Migrated pipeline to .NET 8
	    
	-  Fixed .NET SDK installation
	    
	-  Fixed incorrect `.csproj` paths
	    
	-  Fixed Docker build context
	    
	-  Preserved existing API Dockerfile
	    
	-  Made generic Docker build template repository-aware
	    
	-  Identified API/client image repository mismatch
	    
	-  Corrected intended client pipeline structure
	    
	-  Confirmed separate ACR repositories
	    
	-  Successfully reached Trivy scanning
	    
	-  Confirmed Trivy vulnerability detection
	    
	-  Identified why Trivy stopped ACR push
	    
	-  Decided to temporarily soft-fail Trivy
	    
	-  Deleted AKS to avoid unnecessary cost
	    
	-  Reduced current project scope to CI + ACR
	    
	
	---
	
	# 21. Current State
	
	```text
	GitHub
	  ↓
	Azure DevOps
	  ↓
	.NET 8 Restore
	  ↓
	.NET 8 Build
	  ↓
	Docker Build
	  ↓
	Trivy
	  ↓
	ACR Push
	```
	
	The remaining immediate goal is:
	
	```text
	SUCCESSFUL END-TO-END CI RUN
	```
	
	Specifically:
	
	```text
	Build correct image
	        ↓
	Scan correct image
	        ↓
	Trivy reports vulnerabilities
	        ↓
	Pipeline continues
	        ↓
	Push correct repository
	        ↓
	Verify image exists in ACR
	```
	
	---
	
	# 22. Tomorrow: First Things to Check
	
	## Step 1: Run Shopping API pipeline
	
	Expected:
	
	```text
	shoppingapi:<Build.BuildId>
	shoppingapi:latest
	```
	
	No:
	
	```text
	shoppingclient
	```
	
	should appear anywhere in the API build/scan/push flow.
	
	---
	
	## Step 2: Run Shopping Client pipeline
	
	Expected:
	
	```text
	shoppingclient:<Build.BuildId>
	shoppingclient:latest
	```
	
	No:
	
	```text
	shoppingapi
	```
	
	should appear anywhere in the Client build/scan/push flow.
	
	---
	
	## Step 3: Verify ACR
	
	After successful push:
	
	```bash
	az acr repository list \
	  --name hardikarora \
	  --output table
	```
	
	Expected:
	
	```text
	shoppingapi
	shoppingclient
	```
	
	Then:
	
	```bash
	az acr repository show-tags \
	  --name hardikarora \
	  --repository shoppingapi \
	  --output table
	```
	
	and:
	
	```bash
	az acr repository show-tags \
	  --name hardikarora \
	  --repository shoppingclient \
	  --output table
	```
	
	---
	
	# 23. If the Pipeline Fails Tomorrow
	
	Debug in this order:
	
	```text
	1. YAML parsing
	       ↓
	2. .NET restore
	       ↓
	3. .NET build
	       ↓
	4. Docker build
	       ↓
	5. Image tag
	       ↓
	6. Trivy
	       ↓
	7. ACR service connection
	       ↓
	8. ACR push
	```
	
	Do NOT immediately modify the Dockerfile.
	
	First inspect:
	
	```text
	projectPath
	dockerfilePath
	imageRepository
	buildContext
	containerRegistry
	imageTag
	```
	
	---
	
	# 24. Golden Rule for This Project
	
	For every microservice:
	
	```mermaid
	flowchart LR
	    R[Repository Identity] --> P[Project Path]
	    R --> D[Dockerfile]
	    R --> B[Docker Build]
	    R --> T[Trivy]
	    R --> A[ACR Push]
	```
	
	Everything must point to the **same service**.
	
	### API
	
	```text
	shoppingapi
	Shopping.API
	Shopping.API.csproj
	Shopping.API/Dockerfile
	```
	
	### Client
	
	```text
	shoppingclient
	Shopping.Client
	Shopping.Client.csproj
	Shopping.Client/Dockerfile
	```
	
	If one stage says `shoppingapi` and another says `shoppingclient`, expect:
	
	```text
	tag mismatch
	image not found
	push failure
	```
	
	---
	
	# 25. Resume Point
	
> 	**STOPPED HERE**
	
	The project is now at the point where the CI pipeline should be tested with:
	
	```text
	Trivy = report only
	ACR push = enabled
	AKS = disabled
	```
	
	### Tomorrow's immediate task
	
	```text
	Run API CI
	    ↓
	Verify shoppingapi image
	    ↓
	Run Client CI
	    ↓
	Verify shoppingclient image
	    ↓
	Verify both repositories in ACR
	```
	
	Once both work:
	
	```text
	CI = stable
	```
	
	Then continue with the next DevSecOps layer rather than changing working infrastructure.
	
	---
	
	## Final Mental Model
	
	```mermaid
	flowchart TD
	    G[GitHub] --> AD[Azure DevOps]
	
	    AD --> API[Shopping API Pipeline]
	    AD --> CLIENT[Shopping Client Pipeline]
	
	    API --> APIBUILD[Build shoppingapi]
	    CLIENT --> CLIENTBUILD[Build shoppingclient]
	
	    APIBUILD --> TRIVY1[Trivy]
	    CLIENTBUILD --> TRIVY2[Trivy]
	
	    TRIVY1 --> ACR[Azure Container Registry]
	    TRIVY2 --> ACR
	
	    ACR --> A[shoppingapi]
	    ACR --> C[shoppingclient]
	```
	
	**Finish the end-to-end CI proof first.**
