![[Pasted image 20260902150005.png]]

cat db-deployment.yaml

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: iron-db-deployment-devops
  namespace: iron-namespace-devops
spec:
  replicas: 1
  selector:
    matchLabels:
      db: mariadb
  template:
    metadata:
      labels:
        db: mariadb
    spec:
      containers:
        - name: iron-db-container-devops
          image: kodekloud/irondb:2.0
          env:
            - name: MYSQL_DATABASE
              value: database_web
            - name: MYSQL_ROOT_PASSWORD
              value: "secure@mysqlroot"
            - name: MYSQL_USER
              value: "hardik"
            - name: MYSQL_PASSWORD
              value: "secure@mysql"
          volumeMounts:
            - name: db
              mountPath: /var/lib/mysql
      volumes:
        - name: db
          emptyDir: {}


```


cat gallery-deployment.yaml 

```yaml

apiVersion: apps/v1
kind: Deployment
metadata:
  name: iron-gallery-deployment-devops
  namespace: iron-namespace-devops
spec:
  replicas: 1
  selector:
    matchLabels:
      run: iron-gallery
  template:
    metadata:
      labels:
        run: iron-gallery
    spec:
      containers:
        - name: iron-gallery-container-devops
          image: kodekloud/irongallery:2.0
          resources:
            limits:
              memory: "100Mi"
              cpu: "50m"
          volumeMounts:
            - name: config
              mountPath: /usr/share/nginx/html/data
            - name: images
              mountPath: /usr/share/nginx/html/uploads



      volumes:
        - name: config
          emptyDir: {}
        - name: images
          emptyDir: {}

```

cat svc-app.yaml 
```yaml

apiVersion: v1
kind: Service
metadata:
  name: iron-gallery-service-devops
  namespace: iron-namespace-devops
spec:
  type: NodePort
  selector:
    run: iron-gallery
  ports:
    - protocol: TCP
      port: 80
      targetPort: 80
      nodePort: 32678
      
      
```

thor@jump-host ~$ cat svc-db.yaml
```yaml

apiVersion: v1
kind: Service
metadata:
  name: iron-db-service-devops
  namespace: iron-namespace-devops
spec:
  type: ClusterIP
  selector:
    db: mariadb
  ports:
    - protocol: TCP
      port: 3306
      targetPort: 3306

```

![[Pasted image 20260902150352.png]]
