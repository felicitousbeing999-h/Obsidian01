![[Pasted image 20260901212036.png]]
# redis deployment not getting started, stuck in ContainerCreating status because of wrong 'redis-conig' instead of 'redis-config' and 


```bash
kubectl get logs deployment/redis-deployment
$ something



$ kubectl get events --sort-by='.lastTimestamp'
$ all logs suggesting mistake

$ kubectl edit deployment/redis-deployment


```

vi opens and 
 find `redis:alpin` instead of `redis:alpin`
  and `redis-conig` instead of `redis-config`
```bash
$ kubectl rollout restart deployment/redis-deployment
```

