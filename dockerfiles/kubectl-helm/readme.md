# kubectl-helm Docker Image & OSS‐backed Chart Loader

This repo contains a Dockerfile to build a small Alpine-based image with `kubectl`, Helm, and MinIO client (`mc`) installed, plus Kubernetes manifests to deploy it with the necessary RBAC permissions and OSS support.

## Prerequisites

- Docker (or [Minikube](https://minikube.sigs.k8s.io/docs/) for local testing)
- `kubectl` CLI configured to talk to your cluster
- Helm CLI
- MinIO client (`mc`)
- (Optional) `minikube` for local Kubernetes

## New: external `/charts` volume + OSS support

We added:
- MinIO client (`mc`) for S3/MinIO.
- A Docker `VOLUME ["/charts"]` so you can mount chart-tgzs.
- Kubernetes `Secret` + env vars to configure your OSS credentials.

## Build & Load the Image

### Standard Docker build

```bash
docker build -t your-registry/kubectl-helm:latest .
docker push your-registry/kubectl-helm:latest
```

### Minikube build

If you are using Minikube, you can build the image directly into the Minikube VM:

```bash
minikube start
eval $(minikube docker-env)
docker build -t kubectl-helm:latest .
```

Or you can use the `--load` option to load the image into Minikube:

```bash
docker build -t kubectl-helm:latest .
minikube image load kubectl-helm:latest
```

## Deploy the Image

### 1) Create your OSS & MySQL secrets

```bash
# S3 / MinIO credentials
kubectl create secret generic s3-credentials \
  --from-literal=accesskey=YOUR_ACCESS_KEY \
  --from-literal=secretkey=YOUR_SECRET_KEY

# MySQL credentials (matches your mysql-deployment.yaml)
kubectl create secret generic mysql-credentials \
  --from-literal=username=dbuser \
  --from-literal=password=mysqlpass \
  --from-literal=database=mydb
```

### 2) Deploy with OSS + chart‐mount + MySQL

```bash
kubectl apply -f deploy/k8s-cluster-permissions.yaml
# OR for namespace-scoped RBAC
kubectl apply -f deploy/k8s-deployment.yaml
```

This will create a `kubectl-helm` deployment with the necessary RBAC permissions to run `kubectl`, `helm`, and `mc` commands. The deployment will use the image built in the previous step.

- A service account named `kubectl-helm-admin` will be created.
- A cluster role named `kubectl-helm-cluster-role` will be created with the necessary permissions to run `kubectl`, `helm`, and `mc` commands.
- A cluster role binding named `kubectl-helm-cluster-rolebinding` will be created to bind the service account to the cluster role.  
- A deployment named `kubectl-helm-deployment` will be created with the image built in the previous step.

## Verify the Deployment

### Check resources
To check the resources created, run:

```bash  
kubectl get sa,cr,crb,deploy -n default
```

### Get the pod name and shell in

```bash
POD=$(kubectl get pods -l app=kubectl-helm -o jsonpath='{.items[0].metadata.name}')
kubectl exec -it $POD -- bash
```

### Load a chart into `/charts`

```bash
POD=$(kubectl get pod -l app=kubectl-helm -o jsonpath='{.items[0].metadata.name}')
# configure mc
kubectl exec -it $POD -- mc alias set s3 $MC_HOST_s3 $MC_ACCESS_KEY $MC_SECRET_KEY
# copy chart from bucket
kubectl exec -it $POD -- mc cp s3/my-bucket/mychart-1.2.3.tgz /charts/
# install locally
kubectl exec -it $POD -- helm install mychart /charts/mychart-1.2.3.tgz -n default
```

_No PVs needed – charts live in an `emptyDir`._

### Inside the pod you can run

```bash
# K8s operations
kubectl get pods,services,deployments
kubectl create namespace test-namespace

# Helm operations
helm repo add bitnami https://charts.bitnami.com/bitnami
helm install my-nginx bitnami/nginx -n test-namespace --create-namespace
helm list -A
helm upgrade my-nginx bitnami/nginx -n test-namespace
helm uninstall my-nginx -n test-namespace
```

## Testing on Minikube

### Create a test namespace

```bash
kubectl create namespace test-namespace
```

### Deploy and verify the deployment

```bash
kubectl apply -f deploy/k8s-deployment.yaml
```

### Check the deployment

```bash
kubectl get pods,services,deployments -n test-namespace
```

Use `minikube dashboard` to check the deployment in the dashboard. Forward ports if necessary:

```bash   
kubectl port-forward -n test-namespace svc/kubectl-helm 8080:80
```

## Permission scope

- Cluster-wide: The provided ClusterRole allows full control over namespaces and all core & custom resources.
- Least-privilege: In production, tailor rules to only the resources & verbs you need.

## Cleanup

To delete the deployment and all associated resources, run:

```bash
kubectl delete -f deploy/k8s-deployment.yaml
```

This will remove the deployment, service account, cluster role, and cluster role binding.