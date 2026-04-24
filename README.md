# 🎮 RetroGame K8s Manifests

> Kubernetes manifests for RetroGame Shop | GitOps with ArgoCD | Managed by CI/CD pipeline

## 📋 Overview

This repository contains all Kubernetes manifests for the RetroGame microservices platform. It follows the **GitOps pattern** — ArgoCD watches this repository and automatically deploys any changes to the EKS cluster.

## 🔄 GitOps Flow

Developer pushes code to microservices repo
↓
GitHub Actions CI builds and tests
↓
Docker image pushed to DockerHub
↓
CI automatically updates image tag in THIS repo
↓
ArgoCD detects the commit (within 3 minutes)
↓
ArgoCD applies updated manifests to EKS cluster
↓
Kubernetes performs rolling update
↓
Zero downtime deployment ✅

## 📁 Repository Structure
### Kubernetes Manifests Structure

```text
retrogame-k8s-manifests/
├── frontend/
│   ├── deploy.yaml             # Deployment manifest
│   └── svc.yaml                # Service manifest
├── product-service/
│   ├── deploy.yaml
│   └── svc.yaml
├── cart-service/
│   ├── deploy.yaml
│   └── svc.yaml
├── order-service/
│   ├── deploy.yaml
│   └── svc.yaml
├── payment-service/
│   ├── deploy.yaml
│   └── svc.yaml
└── notification-service/
    ├── deploy.yaml
    └── svc.yaml
```

## 🛠️ Manifest Structure

### Deployment Example
```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: frontend
spec:
  replicas: 1
  selector:
    matchLabels:
      app: frontend
  template:
    spec:
      containers:
      - name: frontend
        image: oghenetejiri798/frontend:latest
        ports:
        - containerPort: 3000
```

### Service Example
```yaml
apiVersion: v1
kind: Service
metadata:
  name: frontend
  namespace: retrogame
spec:
  selector:
    app: frontend
  type: ClusterIP    ← ClusterIP for production (ALB handles external traffic)
  ports:
    - port: 3000
      targetPort: 3000
```

## 🚀 Running Locally
To test manifests locally you need:
- `kubectl` installed
- A running Kubernetes cluster (minikube, kind, or Docker Desktop)

**1. Change service type to NodePort for local access:**
```yaml
spec:
  type: NodePort      # Change from ClusterIP
  ports:
    - port: 3000
      targetPort: 3000
      nodePort: 30000  # Access via localhost:30000
```

**2. Apply manifests:**
```bash
kubectl apply -f frontend/
kubectl apply -f product-service/
kubectl apply -f cart-service/
kubectl apply -f order-service/
kubectl apply -f payment-service/
kubectl apply -f notification-service/
```
**3. Access the app:**
http://localhost:30000

## ☸️ Production Deployment (EKS + ArgoCD)

In production this repo is managed by ArgoCD running on AWS EKS:

| Component | Detail |
|---|---|
| Cluster | AWS EKS (Kubernetes 1.31) |
| Namespace | `retrogame` |
| GitOps Tool | ArgoCD |
| Sync Policy | Automated (every 3 minutes) |
| Load Balancer | AWS ALB (Application Load Balancer) |
| Node Scaling | Karpenter |

### ArgoCD Application Config
```yaml
apiVersion: argoproj.io/v1alpha1
kind: Application
metadata:
  name: retrogame-app
  namespace: argocd
spec:
  source:
    repoURL: https://github.com/Edwin-Oghenetejiri1/retrogame-k8s-manifests.git
    path: .
    directory:
      recurse: true    ← scans all subfolders
  destination:
    namespace: retrogame
  syncPolicy:
    automated:
      prune: true       ← removes deleted resources
      selfHeal: true    ← reverts manual changes
```

## 🌐 Services & Ports

| Service | Port | Type | Description |
|---|---|---|---|
| Frontend | 3000 | ClusterIP | Main UI — only service exposed via ALB |
| Product Service | 8080 | ClusterIP | Product catalog API |
| Cart Service | 8081 | ClusterIP | Shopping cart API |
| Order Service | 8082 | ClusterIP | Order management API |
| Payment Service | 8083 | ClusterIP | Payment processing API |
| Notification Service | 8084 | ClusterIP | Notification API |

> **Note:** All services are ClusterIP in production. Only the frontend is exposed externally via AWS ALB. Backend services communicate internally using Kubernetes DNS (e.g. `http://cart-service:8081`).

## 🔗 Related Repositories

- [RetroGame Microservices](https://github.com/Edwin-Oghenetejiri1/retrogame-microservices-k8s) — Application source code and CI/CD
- [RetroGame EKS Infrastructure](https://github.com/Edwin-Oghenetejiri1/retrogame-eks-infra) — Terraform EKS infrastructure
