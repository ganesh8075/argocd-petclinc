# GitOps-Based Continuous Delivery Using Argo CD on Kubernetes

## 1. Project Overview

This project demonstrates GitOps-based continuous delivery using Argo CD to deploy applications on a Kubernetes cluster.

GitOps is a modern DevOps practice where Git repositories act as the single source of truth for application deployment and infrastructure configuration. Instead of manually deploying applications using kubectl, a GitOps tool such as Argo CD automatically synchronizes the Kubernetes cluster with the desired state defined in Git.

In this project:

- Kubernetes manifests are stored in GitHub  
- Argo CD monitors the Git repository  
- When changes occur in Git, Argo CD automatically deploys them to Kubernetes  

This enables automated, version-controlled, and reliable deployments.

---

## 2. Architecture

The architecture of this project follows the GitOps model.


Dev0ps engineer 
│
│ Push code / Kubernetes manifests
▼
GitHub Repository
(Kubernetes Deployment YAML)
│
│ monitored by
▼
Argo CD
(Running inside Kubernetes cluster)
│
│ Synchronization
▼
Kubernetes Cluster
│
▼
Application Pods running in cluster

### Components Used

| Component   | Purpose                             |
|------------|-------------------------------------|
| Kubernetes | Container orchestration platform     |
| Argo CD    | GitOps continuous delivery tool      |
| GitHub     | Stores Kubernetes manifests          |
| kubectl    | CLI to manage Kubernetes cluster     |

---

## 3. Prerequisites

Before starting the project, the following tools and environment must be available:

### Required Tools

- Kubernetes cluster (Minikube / EKS / Kind)  
- kubectl installed  
- Git installed  
- GitHub repository containing Kubernetes manifests  

### Verify Kubernetes cluster

```bash id="h0l5k2"
kubectl get nodes

```

4. Installing Argo CD

First, create a dedicated namespace for Argo CD.

Create namespace
kubectl create namespace argocd
Set current namespace
kubectl config set-context --current --namespace=argocd

This ensures all commands run inside the argocd namespace.

Install Argo CD using official manifests
```
kubectl apply -n argocd \
-f https://raw.githubusercontent.com/argoproj/argo-cd/stable/manifests/install.yaml
```
This command deploys all Argo CD components into the Kubernetes cluster.

5. Verify Argo CD Installation

Check the running pods:
```

kubectl get pods -n argocd
```

Example output:
```

argocd-server

argocd-repo-server

argocd-application-controller

argocd-dex-server

argocd-redis
```

These components handle:

authentication

Git repository access

deployment synchronization

application management

6. Accessing Argo CD Dashboard

Argo CD provides a web-based dashboard to manage applications.

Check service:
```
kubectl get svc argocd-server -n argocd
```
Use port forwarding:
```
kubectl port-forward svc/argocd-server -n argocd 8080:443
```

Now open the browser:

https://localhost:8080
Default login credentials

Username:

admin

Retrieve password:
```

kubectl get secrets argocd-initial-admin-secret \
-n argocd -o jsonpath="{.data.password}" | base64 -d
```
7. GitHub Repository Setup

Create a GitHub repository that contains Kubernetes manifests.

Example repositories used in the project:
```

https://github.com/ganesh8075/argocd-eapp

https://github.com/ganesh8075/argocd-petclinc
```

Example repository structure:
```
application-repo
│
├── deployment.yaml
└── service.yaml
```
Deployment YAML
```
apiVersion: apps/v1
kind: Deployment
metadata:
  name: ecomm
spec:
  replicas: 2
  selector:
    matchLabels:
      app: ecomm
  template:
    metadata:
      labels:
        app: ecomm
    spec:
      containers:
      - name: ecomm
        image: nginx
        ports:
        - containerPort: 80
```

Service YAML
```
apiVersion: v1
kind: Service
metadata:
  name: ecomm-service
spec:
  selector:
    app: ecomm
  ports:
    - protocol: TCP
      port: 80
      targetPort: 80
  type: ClusterIP
```
8. Creating Application in Argo CD

Inside the Argo CD UI, create a new application.

Application Configuration
Parameter	Value
Application Name	ecommapp
Project	default
Repository URL: GitHub repository
Path	.
Cluster	Kubernetes default cluster
Namespace	default
What this configuration means

Argo CD will:

Pull manifests from the GitHub repository

Compare them with the Kubernetes cluster

Deploy them if they are not present

9. Synchronization Process

Argo CD constantly checks:

Git Repository State
        vs
Kubernetes Cluster State

If they differ:

Argo CD performs synchronization.

Manual Sync

You can click SYNC in the Argo CD dashboard.

Automatic Sync

Argo CD can also deploy changes automatically whenever Git updates.

Example flow:
Developer pushes update to GitHub
        │
        ▼
Argo CD detects change
        │
        ▼
Argo CD deploys update to Kubernetes
10. Verifying Deployment

Check Kubernetes resources:
```

kubectl get deployments
kubectl get pods
kubectl get svc
```
Once pods are running, the application is successfully deployed.

11. Advantages of GitOps with Argo CD
 1. Automated Deployments

Applications deploy automatically when Git changes.

 2. Version Control

All deployment configurations are stored in Git.

 3. Easy Rollback

 If deployment fails:
 
```

 git revert

```

Argo CD automatically restores the previous version.

4. Improved Visibility

Argo CD dashboard shows:

application health

synchronization status

deployment history


