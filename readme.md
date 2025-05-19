# K8s Deployment Cookbook

A collection of “deployment kits” to bootstrap common services and tools in your Kubernetes cluster.  
Each sub-folder is a self-contained example with manifests, Dockerfiles and step-by-step instructions.

## Repository Structure

├── kubectl-helm/  
│   └─ Dockerfile, RBAC & Deployment for a utility pod with `kubectl`, Helm & OSS (MinIO/S3) client.  
│   └─ `readme.md` → How to build the image, create secrets, and run Helm charts from an external bucket.  
│  
├── storages/  
│   └─ `minio-deployment.yaml`, `mysql-deployment.yaml` → PVC, Deployment & Service for MinIO and MySQL.  
│   └─ `readme.md` → How to deploy, access console, connect from in-cluster clients.  
│  
└── readme.md (this file)

## Prerequisites

- A Kubernetes cluster (Minikube, kind, GKE, AKS, etc.)  
- `kubectl` installed & configured  
- Helm 3 CLI  
- Docker (or Minikube’s Docker daemon)

## Quickstart

1. Clone this repo  
   ```bash
   git clone https://github.com/your-org/k8s-deployment-cookbook.git
   cd k8s-deployment-cookbook
   ```

2. Pick a category:

   • **kubectl-helm**  
     └→ `cd kubectl-helm && ./readme.md`  

   • **storages**  
     └→ `cd storages && ./readme.md`  

   Each folder’s README contains build/deploy commands, manifest details and usage examples.

## Adding New Kits

1. Create a new folder under the root (e.g. `logging/`, `monitoring/`, `messaging/`).  
2. Add your YAMLs, Dockerfiles or Helm charts.  
3. Write a concise `readme.md` describing:

   - Purpose & prerequisites  
   - Deployment steps (`kubectl apply`, `helm install`…)  
   - Verification commands  
   - Cleanup instructions  

4. Update this root `readme.md` by adding an entry under **Repository Structure**.

## Support & Contribute

- Issues → https://github.com/your-org/k8s-deployment-cookbook/issues  
- PRs → https://github.com/your-org/k8s-deployment-cookbook/pulls  

Feel free to submit new examples or improvements!<!-- filepath: d:\WSL\repos\temp\composers\dockerfiles\readme.md -->

# K8s Deployment Cookbook

A collection of “deployment kits” to bootstrap common services and tools in your Kubernetes cluster.  
Each sub-folder is a self-contained example with manifests, Dockerfiles and step-by-step instructions.

## Repository Structure

├── kubectl-helm/  
│   └─ Dockerfile, RBAC & Deployment for a utility pod with `kubectl`, Helm & OSS (MinIO/S3) client.  
│   └─ `readme.md` → How to build the image, create secrets, and run Helm charts from an external bucket.  
│  
├── storages/  
│   └─ `minio-deployment.yaml`, `mysql-deployment.yaml` → PVC, Deployment & Service for MinIO and MySQL.  
│   └─ `readme.md` → How to deploy, access console, connect from in-cluster clients.  
│  
└── readme.md (this file)

## Prerequisites

- A Kubernetes cluster (Minikube, kind, GKE, AKS, etc.)  
- `kubectl` installed & configured  
- Helm 3 CLI  
- Docker (or Minikube’s Docker daemon)

## Quickstart

1. Clone this repo  
   ```bash
   git clone https://github.com/your-org/k8s-deployment-cookbook.git
   cd k8s-deployment-cookbook
   ```

2. Pick a category:

   • **kubectl-helm**  
     └→ `cd kubectl-helm && ./readme.md`  

   • **storages**  
     └→ `cd storages && ./readme.md`  

   Each folder’s README contains build/deploy commands, manifest details and usage examples.

## Adding New Kits

1. Create a new folder under the root (e.g. `logging/`, `monitoring/`, `messaging/`).  
2. Add your YAMLs, Dockerfiles or Helm charts.  
3. Write a concise `readme.md` describing:

   - Purpose & prerequisites  
   - Deployment steps (`kubectl apply`, `helm install`…)  
   - Verification commands  
   - Cleanup instructions  

4. Update this root `readme.md` by adding an entry under **Repository Structure**.

## Support & Contribute

- Issues → https://github.com/your-org/k8s-deployment-cookbook/issues  
- PRs → https://github.com/your-org/k8s-deployment-cookbook/pulls  

Feel free to submit new examples or improvements!