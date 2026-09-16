![Pasted image 20260901204040.png](https://github.com/felicitousbeing999-h/Obsidian01/blob/main/Pasted%20image%2020260901204040.png) **Kubernetes batch-style Pod pattern**: inject configuration through environment variables, execute a command, then terminate successfully.

### Mental model

```mermaid
flowchart LR
    K[kubectl apply] --> P[Pod]
    P --> C[ bash container ]
    C --> E[Environment Variables]
    E --> X["/bin/sh -c echo ..."]
    X --> O["Welcome to Nautilus Industries"]
    O --> D[Exit 0]
    D --> F["Pod: Completed"]
```

### What you learned

- **`env:`** → injects runtime configuration into the container.
    
- **`command:`** → overrides the image's default command.
    
- **`/bin/sh -c`** → lets the shell expand `$(GREETING)` etc.
    
- **`restartPolicy: Never`** → don't restart after the command finishes.
    
- **`Completed`** → process exited successfully; **not an error**.
    
- **`kubectl logs`** → retrieves the process's stdout.
    

### Real-world value

This exact pattern appears in:

- **Kubernetes Jobs** → run migrations, backups, cleanup tasks.
    
- **Init containers** → perform setup before the application starts.
    
- **CI/CD** → execute deployment/setup commands.
    
- **Configuration injection** → pass environment-specific values without rebuilding images.
    
- **Debugging** → run a container with a controlled command to inspect behavior.
    

**Platform-engineer takeaway:** Kubernetes doesn't care about your application logic; it manages **process lifecycle + configuration + execution environment**. Here, you explicitly defined all three.