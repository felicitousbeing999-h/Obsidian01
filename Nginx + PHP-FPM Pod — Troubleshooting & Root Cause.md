

> [!abstract] Problem  
> Nginx returned **404 Not Found** even though the Nginx ConfigMap looked correct.
> 
> **Root cause:** Nginx and PHP-FPM mounted the same shared volume at **different filesystem paths**.

![[Pasted image 20260829131306.png]]


---

![[Pasted image 20260829131549.png]]


## 1. Inspect the Nginx Configuration

```bash
kubectl get configmap nginx-config -o yaml
```
![[Pasted image 20260829131644.png]]
**Why:** Validate the configuration actually consumed by Nginx.

Key values:

```yaml
listen 8099;
root /var/www/html;
fastcgi_pass ...
```

### Expected relationship

```mermaid
flowchart LR
    CM["nginx-config"] --> N["Nginx"]
    N --> ROOT["root /var/www/html"]
    ROOT --> PHP["PHP-FPM"]
```

> [!important] Configuration vs Runtime  
> The ConfigMap said Nginx should serve files from `/var/www/html`.  
> Next step: verify that the **container filesystem actually provides that path**.

---

# 2. Inspect the Pod

```bash
kubectl get pods
```

**Why:** Identify the Pod and confirm its runtime state.
![[Pasted image 20260829131846.png]]


```bash
kubectl get pod nginx-php-fpm -o yaml
```

![[Pasted image 20260829131816.png]]
**Why:** Inspect the actual Pod specification, especially:

- `containers`
    
- `volumeMounts`
    
- `volumes`
    
- container names
    

---

## 3. Find the Mount-Path Mismatch

The Pod contains:

```text
nginx-php-fpm
├── nginx
└── php-fpm
```

Both containers use the shared volume:

```yaml
volumes:
- name: shared-files
```

But the mount paths were different.

```mermaid
flowchart TD
    V[shared-files Volume]

    V --> N["Nginx<br/>/different/path"]
    V --> P["PHP-FPM<br/>/var/www/html"]

    N -. "Different filesystem path" .-> X["File visibility mismatch"]
    P -.-> X

    style X stroke-width:2px
```

###  This breaks the application 

A Kubernetes volume is shared at the **storage level**, not automatically at the same filesystem path.

```text
Same volume
      │
      ├── Nginx → /path-A
      │
      └── PHP-FPM → /path-B
```

Therefore:

```text
Nginx sees:     /path-A/index.php
PHP-FPM sees:   /path-B/index.php
```

The containers are effectively looking at **different locations**.

---

# 4. Correct the Mount Path

Export the existing Pod:

```bash
kubectl get pod nginx-php-fpm -o yaml > pod.yaml
```

**Why:** Create a local manifest that can be edited and reapplied.


Edit:

```bash
vi pod.yaml
```

Change the Nginx mount:

```yaml
volumeMounts:
- name: shared-files
  mountPath: /var/www/html
```

![[Pasted image 20260829132143.png]]
Now both containers agree:

![[Pasted image 20260829132345.png]]

```mermaid
flowchart LR
    V["shared-files"]

    V --> N["Nginx<br/>/var/www/html"]
    V --> P["PHP-FPM<br/>/var/www/html"]

    N --> F["Same filesystem location"]
    P --> F

    style F stroke-width:2px
```

> [!success] Correct State  
> **Nginx and PHP-FPM must mount the shared volume at the same logical application path** when both processes need to access the same files.

---

# 5. Recreate the Pod

Delete the old Pod and Apply the corrected manifest:

![[Pasted image 20260829132435.png]]

> **Why:** The Pod needs to be recreated with the corrected `volumeMount`and Creates the Pod using the modified specification.


```mermaid
flowchart LR
    OLD["Broken Pod"] -->|delete| X["Removed"]
    YAML["Corrected pod.yaml"] -->|apply| NEW["New Pod"]
    NEW --> N["Nginx<br/>/var/www/html"]
    NEW --> P["PHP-FPM<br/>/var/www/html"]
```

---

# 6. Copy the Application File

```bash
kubectl cp index.php nginx-php-fpm:/var/www/html/index.php -c nginx
```

### Command anatomy

|Component|Purpose|
|---|---|
|`kubectl cp`|Copy files between local machine and Pod|
|`index.php`|Source file on jump host|
|`nginx-php-fpm:`|Target Pod|
|`/var/www/html/index.php`|Destination inside container|
|`-c nginx`|Select the Nginx container|

> [!important] Why `-c nginx`?  
> The Pod has **two containers**. Explicitly specifying `-c nginx` prevents `kubectl` from selecting the wrong container.

---

# 7. Final Request Path

```mermaid
flowchart LR
    USER["Client"] --> N["Nginx :8099"]
    N --> ROOT["/var/www/html"]
    ROOT --> INDEX["index.php"]
    N -->|FastCGI| PHP["PHP-FPM"]
    PHP --> ROOT
    PHP --> RESP["PHP Response"]
    RESP --> N
    N --> USER
```

---

# Root Cause → Resolution

```mermaid
flowchart TD
    A["404 Not Found"]
    A --> B["Inspect ConfigMap"]
    B --> C["root = /var/www/html"]
    C --> D["Inspect Pod"]
    D --> E["Shared volume found"]
    E --> F["Nginx mountPath ≠ PHP-FPM mountPath"]
    F --> G["Filesystem path mismatch"]
    G --> H["Set both mountPath = /var/www/html"]
    H --> I["Recreate Pod"]
    I --> J["Copy index.php"]
    J --> K["Application accessible"]

    style A stroke-width:2px
    style F stroke-width:2px
    style K stroke-width:2px
```

> [!tip] Architectural Takeaway  
> **Kubernetes volume sharing does not imply path sharing.**
> 
> When sidecar/container processes collaborate on the same application files, verify all three layers:
> 
> **ConfigMap → Container `mountPath` → Application document root**
> 
> They must describe the **same filesystem contract**.