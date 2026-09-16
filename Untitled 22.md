![[Pasted image 20260912021555.png]]![[Pasted image 20260912021652.png]]

```groovy

pipeline {
    agent { label 'stapp01' }

    parameters {
        string(name: 'BRANCH', defaultValue: 'master', description: 'Branch to deploy')
    }

    stages {
        stage('Deploy') {
            steps {
                sh '''
                    cd /var/www/html

                    # Reset and fetch to ensure clean branch checkout
                    git reset --hard
                    git clean -fd
                    git fetch origin

                    if [ "$BRANCH" = "master" ]; then
                        git checkout -B master origin/master
                        git reset --hard origin/master
                    elif [ "$BRANCH" = "feature" ]; then
                        git checkout -B feature origin/feature
                        git reset --hard origin/feature
                    else
                        echo "ERROR: Invalid branch value: $BRANCH"
                        echo "Allowed values are: master or feature"
                        exit 1
                    fi
                '''
            }
        }
    }
}
```

# Dynamic CI/CD Delivery Engine: Architecture & War Room Retrospective

## System Topology & Deployment Flow

Code snippet

```mermaid
graph TD
    User([Dev / Pipeline Trigger]) -->|Build with Parameters<br/>BRANCH=master / feature| JenkinsMaster[Jenkins Controller]
    
    subgraph Agent_Node [stapp01 - App Server 1]
        SSH[OpenSSH Service :22]
        AgentWS[/home/sarah/jenkins_agent/]
        DocRoot[ /var/www/html]
        Apache[Apache HTTPD :8080]
        
        SSH -->|Fork process as user: tony| AgentWS
        AgentWS -->|Remoting Agent JAR execution| DocRoot
        DocRoot -->|Static Files Serviced| Apache
    end

    subgraph Ingress
        Apache --> LB[Application Load Balancer]
        LB --> Public[Public Ingress: Root Path /]
    end

    JenkinsMaster -->|SSH Launcher / SFTP| SSH
```

## Root Cause Analysis: Node Provisioning & Execution Failures

Code snippet

```mermaid
mindmap
  root((Incident Failures))
    Known_Hosts Mismatch
      Missing /var/lib/jenkins/.ssh/known_hosts
      Fix: Non-Verifying Strategy or keyscan preload
    Identity & Permission Friction
      Controller logged in as tony, not sarah
      tony blocked by traverse bits on /home/sarah
      Fix: chmod 755 /home/sarah && chown tony agent root
    Pipeline Workspace Collision
      dir helper allocated /var/www/html@tmp
      Parent /var/www not owned by agent runner
      Fix: Native cd inside durable task step
    Git Workspace Contamination
      Untracked files blocked branch checkout
      Fix: git reset hard and git clean -fd force sweep
```

## Technical Caveman Breakdown

### 1. SSH Host Key Failure

- **Break**: Jenkins want check `known_hosts`. File not on disk. SSH scream and die.
    
      
    
- **Fix**: Flip strategy to `NonVerifyingKeyVerificationStrategy`. Handshake pass.
    
      
    

### 2. SFTP Remoting Drop Failure

- **Break**: User `tony` try write `remoting.jar` inside `/home/sarah/jenkins_agent`. `/home/sarah` locked (`700`). `tony` no enter.
    
      
    
- **Fix**: `chmod 755 /home/sarah`. `chown -R tony:tony /home/sarah/jenkins_agent`. Engine drop jar clean.
    
      
    

### 3. Workspace `@tmp` Directory Trap

- **Break**: Pipeline step `dir('/var/www/html')` make sibling folder `/var/www/html@tmp`. `/var/www` owned by root. Write fail hard.
    
      
    
- **Fix**: No Jenkins `dir()` step. Use native shell: `cd /var/www/html`. Temporary files stay inside agent home directory.
    
      
    

### 4. Git Dirty Tree Block

- **Break**: Branch switch want overwrite local modified files (`feature.html`, `index.html`). Git abort checkout.
    
      
    
- **Fix**: Force wipe before branch shift: `git reset --hard` + `git clean -fd`. Pristine sync every time.
    
      
    

## Core Pipeline Execution State Machine

Code snippet

```mermaid
stateDiagram-v2
    [*] --> Idle
    Idle --> Stage_Deploy: Trigger (BRANCH param)
    
    state Stage_Deploy {
        [*] --> Navigate: cd /var/www/html
        Navigate --> Sanitization: git reset --hard && git clean -fd
        Sanitization --> SyncRemote: git fetch origin
        
        state BranchRoute <<choice>>
        SyncRemote --> BranchRoute
        
        BranchRoute --> DeployMaster: if BRANCH == 'master'
        BranchRoute --> DeployFeature: if BRANCH == 'feature'
        BranchRoute --> Terminate: Invalid branch
        
        DeployMaster --> FastForwardMaster: git checkout -B master origin/master
        DeployFeature --> FastForwardFeature: git checkout -B feature origin/feature
        
        FastForwardMaster --> Complete
        FastForwardFeature --> Complete
        Terminate --> FailState: exit 1
    }

    Complete --> [*]: Apache Serves Docroot
    FailState --> [*]: Build Failure Alert
```

## Canonical Pipeline Artifact



```Groovy
pipeline {
    agent { label 'stapp01' }

    parameters {
        string(name: 'BRANCH', defaultValue: 'master', description: 'Branch target')
    }

    stages {
        stage('Deploy') {
            steps {
                sh '''
                    cd /var/www/html

                    # Purge dirty tree drift
                    git reset --hard
                    git clean -fd
                    git fetch origin

                    # Controlled branch deployment
                    if [ "$BRANCH" = "master" ]; then
                        git checkout -B master origin/master
                        git reset --hard origin/master
                    elif [ "$BRANCH" = "feature" ]; then
                        git checkout -B feature origin/feature
                        git reset --hard origin/feature
                    else
                        echo "FATAL: Invalid branch: $BRANCH"
                        exit 1
                    fi
                '''
            }
        }
    }
}
```

## Architectural Rules of Engagement

- **Never Use Jenkins `dir()` Over System Mounts**: Creates sibling `@tmp` scratchpads. If user lacks root write permissions on parent directory, build fails immediately.
    
      
    
- **Stateless Shell Operations Over Dirty States**: Always enforce hard resets (`git reset --hard`, `git clean -fd`) when continuous delivery agents deploy directly into live web document roots.
    
      
    
- **Boundary Traversal Hygiene**: Owning a sub-folder is useless if intermediate parent folders deny execute (`+x`) permissions to the effective execution UID.