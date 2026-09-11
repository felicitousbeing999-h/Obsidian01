
```yaml

cat > deployment.yaml 
apiVersion: apps/v1
kind: Deployment
metadata:
  name: mysql-deployment
spec:
  replicas: 1
  selector:
    matchLabels:
      app: mysql
  template:
    metadata:
      labels:
        app: mysql
    spec:
      containers:
        - name: mysql-container
          image: mysql:8.0
          ports:
            - containerPort: 3306
          env:
            - name: MYSQL-ROOT-PASSWORD
              valueFrom:
                secretKeyRef:
                  name: mysql-root-pass
                  value: password
            - name: MYSQL_DATABASE
              valueFrom:
                secretKeyRef:
                  name: mysql-db-url
                  value: database
            - name: MYSQL-USER
              valueFrom:
                secretKeyRef:
                  name: mysql-user-pass
                  value: username
            - name: MYSQL_PASSWORD
              valueFrom:
                secretKeyRef:
                  name: mysql-user-pass
                  value: password
		volumeMounts:
			- name: mysql-storage
	          mountPath: /var/lib/mysql
	Volumes:
	 - name: mysql-storage
	   persistentVolumeClaim:
		   claimName: mysql-pv-claim
				
```


![Pasted image 20260903184436.png](https://github.com/felicitousbeing999-h/Obsidian01/blob/main/Pasted%20image%2020260903184436.png)

![Pasted image 20260903184514.png](https://github.com/felicitousbeing999-h/Obsidian01/blob/main/Pasted%20image%2020260903184514.png)Here are all the complete manifests for the task, ready to apply or keep for reference.

  

**1. Secrets**

  

You can create them directly via declarative YAML:

  

YAML

```
cat << 'EOF' > secrets.yaml
apiVersion: v1
kind: Secret
metadata:
  name: mysql-root-pass
type: Opaque
stringData:
  password: "YUIidhb667"
---
apiVersion: v1
kind: Secret
metadata:
  name: mysql-user-pass
type: Opaque
stringData:
  username: "kodekloud_joy"
  password: "ksH85UJjhb"
---
apiVersion: v1
kind: Secret
metadata:
  name: mysql-db-url
type: Opaque
stringData:
  database: "kodekloud_db4"
EOF
```

**2. PersistentVolume and PersistentVolumeClaim**

  

YAML

```
cat << 'EOF' > storage.yaml
apiVersion: v1
kind: PersistentVolume
metadata:
  name: mysql-pv
spec:
  capacity:
    storage: 250Mi
  accessModes:
    - ReadWriteOnce
  persistentVolumeReclaimPolicy: Retain
  hostPath:
    path: /mnt/data
---
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: mysql-pv-claim
spec:
  accessModes:
    - ReadWriteOnce
  resources:
    requests:
      storage: 250Mi
EOF
```

**3. Deployment**

  

YAML

```
cat << 'EOF' > deployment.yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: mysql-deployment
spec:
  replicas: 1
  selector:
    matchLabels:
      app: mysql
  template:
    metadata:
      labels:
        app: mysql
    spec:
      containers:
        - name: mysql-container
          image: mysql:8.0
          ports:
            - containerPort: 3306
          env:
            - name: MYSQL_ROOT_PASSWORD
              valueFrom:
                secretKeyRef:
                  name: mysql-root-pass
                  key: password
            - name: MYSQL_DATABASE
              valueFrom:
                secretKeyRef:
                  name: mysql-db-url
                  key: database
            - name: MYSQL_USER
              valueFrom:
                secretKeyRef:
                  name: mysql-user-pass
                  key: username
            - name: MYSQL_PASSWORD
              valueFrom:
                secretKeyRef:
                  name: mysql-user-pass
                  key: password
          volumeMounts:
            - name: mysql-storage
              mountPath: /var/lib/mysql
      volumes:
        - name: mysql-storage
          persistentVolumeClaim:
            claimName: mysql-pv-claim
EOF
```

**4. Service (NodePort: 30007)**

YAML

```
cat << 'EOF' > service.yaml
apiVersion: v1
kind: Service
metadata:
  name: mysql
spec:
  type: NodePort
  selector:
    app: mysql
  ports:
    - port: 3306
      targetPort: 3306
      nodePort: 30007
EOF
```