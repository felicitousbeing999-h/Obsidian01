

1. **Symptoms**
    
    - `curl http://<GATEWAY_URL>/productpage` → Connection refused, later `503 Service Unavailable`.
        
    - Ingress gateway pods were in `ImagePullBackOff`.
        
2. **Root Cause Discovery**
    
    - `kubectl describe pod` showed:
        
        ```
        Image: auto
        Failed to pull image "auto"
        ```
        
    - `auto` is a **placeholder**, not a real image. It must be replaced by the Cloud Service Mesh injection webhook.
        
3. **Why `auto` wasn't replaced**
    
    - `kubectl get mutatingwebhookconfigurations` showed **no Istio/ASM webhook**.
        
    - `gcloud container fleet mesh describe` showed:
        
        ```
        REVISION_PROVISIONING
        ```
        
    - `kubectl get controlplanerevision` showed:
        
        ```
        RECONCILED=False
        ```
        
    - `kubectl get pods -n istio-system` returned **no control plane pods**.
        
4. **Diagnosis**
    
    - The managed Cloud Service Mesh control plane had **not finished provisioning**, so:
        
        - No injection webhook existed.
            
        - Gateway pods tried to pull `docker.io/library/auto:latest`.
            
        - Ingress gateway never started.
            
5. **Resolution**
    
    - Waited until:
        
        ```
        REVISION_READY
        RECONCILED=True
        ```
        
    - Recreated/restarted the ingress gateway pods.
        
    - Gateway became `1/1 Running`.
        
6. **Second Issue**
    
    - `curl` returned:
        
        ```
        HTTP/1.1 503 Service Unavailable
        ```
        
    - This indicated Envoy was running but could not route traffic while the mesh and workloads finished reconciling.
        
7. **Final Verification**
    
    - After the control plane became active and the gateway stabilized:
        
        ```
        curl -I http://<GATEWAY_URL>/productpage
        ```
        
        returned:
        
        ```
        HTTP/1.1 200 OK
        ```
        
    - This confirmed:
        
        - ✅ Managed Cloud Service Mesh was active.
            
        - ✅ Ingress gateway was healthy.
            
        - ✅ Traffic routing was working.
            
        - ✅ Bookinfo application was successfully accessible.
            

**Key lesson:** In managed Cloud Service Mesh, **`image: auto` is expected**. If it results in `ImagePullBackOff`, the issue is usually that the **mesh control plane and injection webhook are not yet ready (`REVISION_PROVISIONING`)**. Wait for **`REVISION_READY`** before deploying or recreating ingress/workloads.