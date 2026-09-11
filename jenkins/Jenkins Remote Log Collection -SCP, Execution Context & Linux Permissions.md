![[Pasted image 20260910215142.png]]
> [!NOTE] Architecture  
> Jenkins executes the transfer command **on the Storage Server**, which then pulls Apache logs from App Server 3.

```mermaid
flowchart LR
    J[Jenkins Controller]
    S[Storage Server<br/>ststor01]
    A[App Server 3<br/>stapp03]
    D[ /usr/src/data]

    J -->|SSH / Publish Over SSH| S
    S -->|scp pull| A
    A -->|/var/log/httpd/<br/>access_log + error_log| S
    S --> D
```

---

## 1. Execution Context

```mermaid
sequenceDiagram
    participant J as Jenkins
    participant S as ststor01
    participant A as stapp03

    J->>S: SSH
    J->>S: Execute sshpass + scp
    S->>A: SSH/SCP as banner
    A-->>S: Apache log
    S->>S: Write /usr/src/data/access_log
```

### Critical rule

```text
SSH Server = Storage Server
             ↓
Exec command runs HERE
             ↓
scp source = stapp03
scp target = local Storage Server
```

---

## 2. Command Semantics

```bash
sshpass -p "BigGr33n" \
scp -o StrictHostKeyChecking=no \
banner@stapp03:/var/log/httpd/access_log \
/usr/src/data/access_log
```

|Component|Role|
|---|---|
|`sshpass`|Non-interactive password injection|
|`scp`|Secure file copy|
|`banner@stapp03:`|Remote source|
|`/var/log/httpd/access_log`|Apache log|
|`/usr/src/data/access_log`|**Local destination**|
|`StrictHostKeyChecking=no`|Avoid host-key prompt|

The destination path is interpreted **on `ststor01`**.

---

## 3. Failure Chain

### Failure ① — Exit `127`

```text
Jenkins
  ↓
Storage Server
  ↓
sshpass
  ↓
NOT FOUND
  ↓
exit 127
```

```bash
which sshpass
# /usr/bin/sshpass
```

**Root cause:** executable unavailable in remote execution environment.

---

### Failure ② — No destination

```text
scp
 ↓
/usr/src/data/access_log
 ↓
directory missing
 ↓
No such file or directory
```

Fix:

```bash
sudo mkdir -p /usr/src/data
```

---

### Failure ③ — Permission denied

```mermaid
flowchart TD
    U[natasha]
    D[ /usr/src/data ]
    R[root:root]

    U -->|write| D
    D -->|owned by| R
    U -.->|DENIED| D
```

Observed:

```text
drwxr-xr-x root root /usr/src/data
```

`natasha`:

```text
r-x
```

but needs:

```text
write
```

Fix:

```bash
sudo chown natasha:natasha /usr/src/data
```

Expected:

```text
drwxr-xr-x natasha natasha /usr/src/data
```

---

## 4. Permission Model

```mermaid
flowchart LR
    N[natasha]
    D[ /usr/src/data]
    F[access_log]
    
    N -->|create/write| D
    D -->|contains| F
```

Required capability:

```text
Directory:
    execute → traverse
    write   → create/delete files
```

`755 root:root`:

```text
root  → rwx
group → r-x
other → r-x
```

Therefore:

```text
natasha ≠ owner
natasha ∉ root group
→ cannot create file
```

---

## 5. Correct State

```mermaid
flowchart TD
    J[Jenkins]
    S[ststor01]
    P[ /usr/src/data]
    AL[access_log]
    EL[error_log]
    A[stapp03]

    J -->|SSH| S
    S -->|scp| A
    A -->|access_log| S
    A -->|error_log| S
    S --> P
    P --> AL
    P --> EL
```

Storage Server:

```bash
ls -ld /usr/src/data
```

Expected:

```text
drwxr-xr-x natasha natasha /usr/src/data
```

Verification:

```bash
ls -lh /usr/src/data/
```

Expected:

```text
access_log
error_log
```

---

## 6. Jenkins Configuration

### Schedule

```text
*/6 * * * *
```

### SSH target

```text
Storage Server
```

### Exec

```bash
sshpass -p "BigGr33n" scp -o StrictHostKeyChecking=no banner@stapp03:/var/log/httpd/access_log /usr/src/data/access_log

sshpass -p "BigGr33n" scp -o StrictHostKeyChecking=no banner@stapp03:/var/log/httpd/error_log /usr/src/data/error_log
```

---

## 7. Our Diagnostic Method

```mermaid
flowchart TD
    E[Jenkins failure]
    C{Can Jenkins SSH<br/>to target?}
    X{Command exists?}
    D{Destination exists?}
    P{Can remote user<br/>write destination?}
    N{Can source be read?}
    OK[Transfer succeeds]

    E --> C
    C -->|No| SSH[Fix SSH configuration]
    C -->|Yes| X
    X -->|No| CMD[Install/fix executable]
    X -->|Yes| D
    D -->|No| DIR[Create directory]
    D -->|Yes| P
    P -->|No| PERM[Fix ownership/ACL]
    P -->|Yes| N
    N -->|No| SRC[Fix source access]
    N -->|Yes| OK
```

### Debug in this order

```bash
whoami
hostname
command -v sshpass
command -v scp
ls -ld /usr/src/data
touch /usr/src/data/test
ls -l /var/log/httpd/
```

---

## 8. Production Architecture ≠ Lab Architecture

### Lab

```text
Jenkins
   ↓
sshpass + password
   ↓
Storage Server
   ↓
scp
```

### Production

```mermaid
flowchart LR
    J[Jenkins]
    C[Credential Store]
    S[Storage]
    A[App]
    L[Central Logging]

    J --> C
    J -->|short-lived identity| S
    S -->|agent / secure transport| L
    A -->|structured logs| L
```

Avoid:

```markdown
- passwords in Jenkinsfile
- passwords in shell commands
- sshpass
- StrictHostKeyChecking=no
- manually distributing log files
```

Prefer:

```markdown
- SSH keys / workload identity
- Jenkins Credentials
-  host-key verification
- centralized logging
- least privilege
- immutable log pipeline
```

---

## 9. Staff-Level Takeaway

> **Always identify the execution context before debugging a remote command.**

```text
Jenkins target
      ↓
Where does Exec execute?
      ↓
What user?
      ↓
What filesystem?
      ↓
What binaries?
      ↓
What permissions?
      ↓
What network path?
      ↓
What source identity?
```

The three failures were three different layers:

```text
127
 ↓
PROCESS / DEPENDENCY

No such file
 ↓
FILESYSTEM / PATH

Permission denied
 ↓
AUTHORIZATION / UNIX PERMISSIONS
```

That separation is the important part.

#jenkins #linux