# Storage Services on Kubernetes: MinIO & MySQL

This guide walks you through deploying MinIO (S3-compatible OSS) and MySQL on a Kubernetes cluster (e.g. Minikube). It also shows how to install and use `kubectl` and Helm for further operations.

## Prerequisites

- A running Kubernetes cluster (Minikube, kind, GKE, AKS, etc.)
- `kubectl` CLI configured to talk to your cluster  
  Install:  
  ```bash
  # Linux / Mac
  curl -LO "https://dl.k8s.io/release/$(curl -L -s https://dl.k8s.io/release/stable.txt)/bin/linux/amd64/kubectl"
  chmod +x kubectl && mv kubectl /usr/local/bin/

  # Windows (PowerShell)
  choco install kubernetes-cli
  ```
- Helm CLI  
  Install:  
  ```bash
  curl https://raw.githubusercontent.com/helm/helm/master/scripts/get-helm-3 | bash
  ```
- (Optional) Minikube for local testing  
  ```bash
  curl -LO https://storage.googleapis.com/minikube/releases/latest/minikube-linux-amd64
  chmod +x minikube-linux-amd64 && mv minikube-linux-amd64 /usr/local/bin/minikube
  minikube start
  ```

---

## 1. Deploy MinIO OSS

Apply **minio-deployment.yaml**:

```bash
kubectl apply -f minio-deployment.yaml
# To delete:
kubectl delete -f minio-deployment.yaml
```

Verify:

```bash
kubectl get pvc,deploy,svc -l app=minio
```
You should see something like this:
```
NAME            TYPE        CLUSTER-IP       EXTERNAL-IP   PORT(S)          AGE
kubernetes      ClusterIP   10.96.0.1        <none>        443/TCP          6d2h
minio-api       ClusterIP   10.108.218.160   <none>        9000/TCP         14m
minio-console   NodePort    10.103.175.211   <none>        9001:30901/TCP   14m
```
Get the MinIO endpoints:

You can use port-forwarding or access via NodePort:
```bash
# Port-forwarding
kubectl port-forward svc/minio-api 9000:9000
kubectl port-forward svc/minio-console 9001:9001
```

Or you can access via `minikube service` command:
```bash
# Access via NodePort
MINIO_API=$(minikube service minio-api --url)
MINIO_CONSOLE=$(minikube service minio-console --url)
echo "API: $MINIO_API"      # e.g. http://192.168.99.100:30000
echo "Console: $MINIO_CONSOLE"  # e.g. http://192.168.99.100:30901
```

Log in to the console with `minioadmin` / `minioadmin`.

---

## 2. Deploy MySQL

Apply **mysql-deployment.yaml**:


```bash
kubectl apply -f mysql-deployment.yaml
# To delete:
kubectl delete -f mysql-deployment.yaml
```

Verify:

```bash
kubectl get pvc,deploy,svc -l app=mysql
```

Connect inside cluster:

```bash
kubectl run mysql-client --rm -i --tty --image=mysql:8.0 -- bash
# inside pod:
mysql -h mysql -u dbuser -pmydb
```

---

## 3. Next Steps

1. **Integrate with `kubectl-helm` image**  
   - Create OSS & MySQL secrets in your cluster:  
     ```bash
     kubectl create secret generic s3-credentials \
       --from-literal=accesskey=minioadmin \
       --from-literal=secretkey=minioadmin

     kubectl create secret generic mysql-credentials \
       --from-literal=username=dbuser \
       --from-literal=password=mysqlpass \
       --from-literal=database=mydb
     ```
   - Refer to the [kubectl-helm README](../kubectl-helm/readme.md) for deployment and usage.

2. **Backup & Maintenance**  
   - MinIO: use `mc` or `aws s3` CLI to mirror buckets.  
   - MySQL: schedule `mysqldump` via CronJob.

3. **Troubleshooting**  
   - Pods stuck? `kubectl describe pod …` + `kubectl logs`.  
   - PVC pending? Ensure your cluster has a default StorageClass.

---

You now have a local S3-compatible object store and a MySQL database running on Kubernetes, ready to be consumed by your applications or your `kubectl-helm` utility container.<!-- filepath: d:\WSL\repos\temp\composers\dockerfiles\storages\readme.md -->

# Storage Services on Kubernetes: MinIO & MySQL

This guide walks you through deploying MinIO (S3-compatible OSS) and MySQL on a Kubernetes cluster (e.g. Minikube). It also shows how to install and use `kubectl` and Helm for further operations.

## Prerequisites

- A running Kubernetes cluster (Minikube, kind, GKE, AKS, etc.)
- `kubectl` CLI configured to talk to your cluster  
  Install:  
  ```bash
  # Linux / Mac
  curl -LO "https://dl.k8s.io/release/$(curl -L -s https://dl.k8s.io/release/stable.txt)/bin/linux/amd64/kubectl"
  chmod +x kubectl && mv kubectl /usr/local/bin/

  # Windows (PowerShell)
  choco install kubernetes-cli
  ```
- Helm CLI  
  Install:  
  ```bash
  curl https://raw.githubusercontent.com/helm/helm/master/scripts/get-helm-3 | bash
  ```
- (Optional) Minikube for local testing  
  ```bash
  curl -LO https://storage.googleapis.com/minikube/releases/latest/minikube-linux-amd64
  chmod +x minikube-linux-amd64 && mv minikube-linux-amd64 /usr/local/bin/minikube
  minikube start
  ```

---

## 1. Deploy MinIO OSS

Save the following manifest as **minio-deployment.yaml**:

```yaml
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: minio-pvc
spec:
  accessModes:
    - ReadWriteOnce
  resources:
    requests:
      storage: 10Gi
---
apiVersion: apps/v1
kind: Deployment
metadata:
  name: minio
spec:
  selector:
    matchLabels:
      app: minio
  template:
    metadata:
      labels:
        app: minio
    spec:
      containers:
      - name: minio
        image: minio/minio:latest
        args:
        - server
        - /data
        - --console-address
        - ":9001"
        env:
        - name: MINIO_ROOT_USER
          value: "minioadmin"
        - name: MINIO_ROOT_PASSWORD
          value: "minioadmin"
        ports:
        - containerPort: 9000
          name: api
        - containerPort: 9001
          name: console
        volumeMounts:
        - name: storage
          mountPath: /data
      volumes:
      - name: storage
        persistentVolumeClaim:
          claimName: minio-pvc
---
apiVersion: v1
kind: Service
metadata:
  name: minio-api
spec:
  ports:
  - port: 9000
    targetPort: 9000
  selector:
    app: minio
---
apiVersion: v1
kind: Service
metadata:
  name: minio-console
spec:
  type: NodePort
  ports:
  - port: 9001
    targetPort: 9001
    nodePort: 30901
  selector:
    app: minio
```

Apply it:

```bash
kubectl apply -f minio-deployment.yaml
```

Verify:

```bash
kubectl get pvc,deploy,svc -l app=minio
```

Get the MinIO endpoints:

```bash
MINIO_API=$(minikube service minio-api --url)
MINIO_CONSOLE=$(minikube service minio-console --url)
echo "API: $MINIO_API"      # e.g. http://192.168.99.100:30000
echo "Console: $MINIO_CONSOLE"  # e.g. http://192.168.99.100:30901
```

Log in to the console with `minioadmin` / `minioadmin`.

---

## 2. Deploy MySQL

Save the following manifest as **mysql-deployment.yaml**:

```yaml
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: mysql-pvc
spec:
  accessModes:
    - ReadWriteOnce
  resources:
    requests:
      storage: 10Gi
---
apiVersion: v1
kind: Secret
metadata:
  name: mysql-secret
type: Opaque
stringData:
  mysql-root-password: mysqlrootpass
  mysql-user:       dbuser
  mysql-password:   mysqlpass
  mysql-database:   mydb
---
apiVersion: apps/v1
kind: Deployment
metadata:
  name: mysql
spec:
  selector:
    matchLabels:
      app: mysql
  template:
    metadata:
      labels:
        app: mysql
    spec:
      containers:
      - name: mysql
        image: mysql:8.0
        env:
        - name: MYSQL_ROOT_PASSWORD
          valueFrom:
            secretKeyRef:
              name: mysql-secret
              key: mysql-root-password
        - name: MYSQL_USER
          valueFrom:
            secretKeyRef:
              name: mysql-secret
              key: mysql-user
        - name: MYSQL_PASSWORD
          valueFrom:
            secretKeyRef:
              name: mysql-secret
              key: mysql-password
        - name: MYSQL_DATABASE
          valueFrom:
            secretKeyRef:
              name: mysql-secret
              key: mysql-database
        ports:
        - containerPort: 3306
          name: mysql
        volumeMounts:
        - name: data
          mountPath: /var/lib/mysql
      volumes:
      - name: data
        persistentVolumeClaim:
          claimName: mysql-pvc
---
apiVersion: v1
kind: Service
metadata:
  name: mysql
spec:
  ports:
  - port: 3306
    targetPort: 3306
  selector:
    app: mysql
```

Apply it:

```bash
kubectl apply -f mysql-deployment.yaml
```

Verify:

```bash
kubectl get pvc,deploy,svc -l app=mysql
```

Connect inside cluster:

```bash
kubectl run mysql-client --rm -i --tty --image=mysql:8.0 -- bash
# inside pod:
mysql -h mysql -u dbuser -pmydb
```

---

## 3. Next Steps

1. **Integrate with `kubectl-helm` image**  
   - Create OSS & MySQL secrets in your cluster:  
     ```bash
     kubectl create secret generic s3-credentials \
       --from-literal=accesskey=minioadmin \
       --from-literal=secretkey=minioadmin

     kubectl create secret generic mysql-credentials \
       --from-literal=username=dbuser \
       --from-literal=password=mysqlpass \
       --from-literal=database=mydb
     ```
   - Refer to the [kubectl-helm README](../kubectl-helm/readme.md) for deployment and usage.

2. **Backup & Maintenance**  
   - MinIO: use `mc` or `aws s3` CLI to mirror buckets.  
   - MySQL: schedule `mysqldump` via CronJob.

3. **Troubleshooting**  
   - Pods stuck? `kubectl describe pod …` + `kubectl logs`.  
   - PVC pending? Ensure your cluster has a default StorageClass.

---

You now have a local S3-compatible object store and a MySQL database running on Kubernetes, ready to be consumed by your applications or your `kubectl-helm` utility container.