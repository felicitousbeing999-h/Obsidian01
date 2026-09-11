A DaemonSet is a Kubernetes resource that ensures that all nodes run a copy of a Pod. Whenever a new node is added to the cluster, Kubernetes will automatically attempt to schedule a DaemonSet Pod onto it.
![[Pasted image 20260904165816.png]]
Typical use cases for DaemonSets include:

- Log collection: agents like Fluentd or Filebeat
- Monitoring: node exporters, metrics collectors
- Networking: CNI plugins, proxies, or sidecar-like agents

Think of a DaemonSet as a Deployment for nodes. Instead of saying "I want 3 replicas," you're saying, "I want one Pod per node." It scales with your infrastructure - not your desired replica count.

Deployment:

- You control the replica count
- 3 pods spread across nodes

DaemonSet:

- You control node coverage
- 1 pod on every node
![[Pasted image 20260904171456.png]]
---
![[Pasted image 20260904171602.png]]
###### Creating a DaemonSet from a Deployment

The best way to understand DaemonSets is by getting hands on - we're going to create our own. The great news is, as you already know how to create a Deployment, you're 90% of the way towards creating DaemonSets.![[Pasted image 20260904171633.png]]

Unlike a Deployment, there isn't a convenient kubectl command for running or creating a DaemonSet, but we can fast track these by using the Deployment spec as our base.

Let's prepare the following command, as if we were creating a Deployment. This deployment is going to run a while loop which outputs the date and hello from the Kubernetes node in which the pod is running. We'll output this to a file with tee -
```bash

kubectl create deployment logger --image=alpine -o yaml --dry-run=client -- /bin/sh -c "while true; do date +\"%Y-%m-%d-%H:%M:%S - Hello from \$NODE_NAME\"; sleep 30; done" | tee logger.yaml
```
Let's run this as a Deployment first. Just to caveat - the NODE_NAME variable doesn't exist yet so the output won't show the node name. Let's apply the yaml -

```bash
kubectl apply -f logger.yaml
```
And then let's get the logs. We'll use the label selector to get logs from every pod associated with this deployment -

```bash
kubectl logs -l app=logger
```
At the moment, we don't know which node this is actually running from. After a pod has been scheduled, a lot more information is populated in the spec. What we are interested in is the spec.nodeName field that will correlate to the node in which the pod is running -
```bash

kubectl get pods
```
Let's take a look at the full pod YAML to see the nodeName field (use the pod name from the output above) -
```bash

kubectl get pod -l app=logger -o yaml | more
```
---

###### Adding the Node Name Environment Variable

We'll delete this current deployment -

```bash
kubectl delete -f logger.yaml --now
```
And we'll modify our yaml. We'll add an env which tells the pod to set an environment variable of NODE_NAME and we are going to use a spec value from the pod, using a fieldRef to spec.nodeName -
```yaml
cat <<EOF > logger.yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  labels:
    app: logger
  name: logger
spec:
  replicas: 1
  selector:
    matchLabels:
      app: logger
  strategy: {}
  template:
    metadata:
      labels:
        app: logger
    spec:
      containers:
      - command:
        - /bin/sh
        - -c
        - while true; do date +"%Y-%m-%d-%H:%M:%S - Hello from \$NODE_NAME"; sleep 30; done
        image: alpine
        name: alpine
        env:
        - name: NODE_NAME
          valueFrom:
            fieldRef:
              fieldPath: spec.nodeName
EOF
```


We'll save and apply this -

```bash
kubectl apply -f logger.yaml
```
And now, if we check the logs -

```bash
kubectl logs -l app=logger | sort
```
That's looking good - we have the node from which it's running in the logs.

---

###### The Problem with Deployments for Node Coverage

Now, we know we have 3 nodes. What happens if we scale to 3 replicas? We might get lucky, we might get one on each node, let's try it -
![[Pasted image 20260904172651.png]]
`kubectl scale --replicas=3 deployment/logger`

Check the logs -

```bash
kubectl logs -l app=logger
```
And get pods also shows the distribution -
```bash
kubectl get pods -o wide
```
You may or may not get lucky when you try this - the distribution and scheduling is completely variable with Deployments. Let's delete this deployment and start working on converting it to a DaemonSet, which is the ideal type when you need a pod per node -

```shell
kubectl delete -f logger.yaml --now
```

---


###### Converting to a DaemonSet

We'll modify the yaml. Three changes are needed:

- Change the kind from Deployment to DaemonSet
- Remove the replicas line (there is no concept of replicas with DaemonSets)
- Remove the strategy line (there is no concept of strategy like rollingUpdate)
  

  
```yaml

cat <<EOF > logger.yaml
apiVersion: apps/v1
kind: DaemonSet
metadata:
  labels:
    app: logger
  name: logger
spec:
  selector:
    matchLabels:
      app: logger
  template:
    metadata:
      labels:
        app: logger
    spec:
      containers:
      - command:
        - /bin/sh
        - -c
        - while true; do date +"%Y-%m-%d-%H:%M:%S - Hello from \$NODE_NAME"; sleep 30; done
        image: alpine
        name: alpine
        env:
        - name: NODE_NAME
          valueFrom:
            fieldRef:
              fieldPath: spec.nodeName
EOF
```

And now, we can apply this -
```shell

kubectl apply -f logger.yaml
```
And voila, we have a DaemonSet! If you check, you can see this running. This will run on all nodes that it possibly can - if you ever don't see the desired number of nodes, it's most likely something like a taint which is stopping it running on a particular node -
```bash

kubectl get daemonset
```
Also be aware of the pod naming - it looks different to a Deployment because there is no ReplicaSet. Recall, Deployment pods are named with the ReplicaSet hash as part of the name -
```bash

kubectl get pods -o wide
```
And if we check the logs, all is as expected -
```bash
kubectl logs -l app=logger | sort
```
With this setup, if we were to add any new nodes to this cluster, that DaemonSet would automatically provision pods to those nodes!

---


#### Cleanup

And lastly, let's clean up -

```bash
kubectl delete -f logger.yaml --now
rm logger.yaml
```