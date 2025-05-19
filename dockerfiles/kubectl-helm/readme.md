# kubectl-helm Docker Image & Kubernetes Deployment

This repo contains a Dockerfile to build a small Alpine-based image with `kubectl` and Helm installed, plus a single K8s manifest (`deploy/kubectl-helm-deploy.yaml`) to deploy it with the necessary RBAC permissions.

## Prerequisites

- Docker (or [Minikube](https://minikube.sigs.k8s.io/docs/) for local testing)
- `kubectl` CLI configured to talk to your cluster
- Helm CLI
- (Optional) `minikube` for local Kubernetes

## Build & Load the Image

### Standard Docker build

```sh
docker build -t your-registry/kubectl-helm:latest .
docker push your-registry/kubectl-helm:latest
```
### Minikube build

If you are using Minikube, you can build the image directly into the Minikube VM:

```sh
minikube start
eval $(minikube docker-env)
docker build -t kubectl-helm:latest .
```

Or you can use the `--load` option to load the image into Minikube:

```sh
docker build -t kubectl-helm:latest .
minikube image load kubectl-helm:latest
```
## Deploy the Image
Apply the combined RBAC and deployment manifest:

```sh
kubectl apply -f deploy/kubectl-helm-deploy.yaml
```
This will create a `kubectl-helm` deployment with the necessary RBAC permissions to run `kubectl` and `helm` commands. The deployment will use the image built in the previous step.

- A service account named `kubectl-helm` will be created.
- A cluster role named `kubectl-helm` will be created with the necessary permissions to run `kubectl` and `helm` commands.
- A cluster role binding named `kubectl-helm` will be created to bind the service account to the cluster role.  
- A deployment named `kubectl-helm` will be created with the image built in the previous step.

## Verify the Deployment

### Check resources
To check the resources created, run:

```sh  
kubectl get sa,cr,crb,deploy -n default
```

### Get the pod name and shell in

```sh
POD=$(kubectl get pods -l app=kubectl-helm -o jsonpath='{.items[0].metadata.name}')
kubectl exec -it $POD -- bash
```
### inside the pod you can run

```sh
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
## Testing on  Minikube
### Create a test namespace
```sh
kubectl create namespace test-namespace
```
### Deploy and verify the deployment
```sh
kubectl apply -f deploy/kubectl-helm-deploy.yaml
```
### Check the deployment
```sh
kubectl get pods,services,deployments -n test-namespace
```
use `minikube dashboard` to check the deployment in the dashboard
forward ports if necessary
```sh   
kubectl port-forward -n test-namespace svc/kubectl-helm 8080:80
```
## Permission scope
- Cluster-wide: The provided ClusterRole allows full control over namespaces and all core & custom resources.
- Least-privilege: In production, tailor rules to only the resources & verbs you need.

```yaml

```yaml
apiVersion: v1
kind: ServiceAccount
metadata:
  name: kubectl-helm-admin
  namespace: default
---
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRole
metadata:
  name: kubectl-helm-cluster-role
rules:
- apiGroups: [""]
  resources:
    - namespaces
    - pods
    - services
    - configmaps
    - secrets
    - persistentvolumes
    - persistentvolumeclaims
  verbs: ["*"]
- apiGroups: ["apps"]
  resources:
    - deployments
    - replicasets
    - statefulsets
    - daemonsets
  verbs: ["*"]
- apiGroups: ["batch"]
  resources:
    - jobs
    - cronjobs
  verbs: ["*"]
- apiGroups: ["networking.k8s.io"]
  resources:
    - ingresses
  verbs: ["*"]
- apiGroups: ["rbac.authorization.k8s.io"]
  resources:
    - roles
    - rolebindings
    - clusterroles
    - clusterrolebindings
  verbs: ["*"]
---
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRoleBinding
metadata:
  name: kubectl-helm-cluster-rolebinding
subjects:
- kind: ServiceAccount
  name: kubectl-helm-admin
  namespace: default
roleRef:
  kind: ClusterRole
  name: kubectl-helm-cluster-role
  apiGroup: rbac.authorization.k8s.io
---
apiVersion: apps/v1
kind: Deployment
metadata:
  name: kubectl-helm-deployment
  namespace: default
spec:
  replicas: 1
  selector:
    matchLabels:
      app: kubectl-helm
  template:
    metadata:
      labels:
        app: kubectl-helm
    spec:
      serviceAccountName: kubectl-helm-admin
      containers:
      - name: kubectl-helm
        image: your-registry/kubectl-helm:latest
        imagePullPolicy: Always
        command: ["bash", "-c", "sleep infinity"]
        resources:
          limits:
            cpu: "0.5"
            memory: "512Mi"
          requests:
            cpu: "0.2"
            memory: "256Mi"
```

## Cleanup
To delete the deployment and all associated resources, run:

```sh
kubectl delete -f deploy/kubectl-helm-deploy.yaml
```
This will remove the deployment, service account, cluster role, and cluster role binding.