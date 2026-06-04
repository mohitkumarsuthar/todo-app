# 🚀 Production-Grade DevOps Platform on AWS EKS

A complete end-to-end DevOps pipeline implementing GitOps, CI/CD automation, container security scanning, and real-time monitoring on AWS EKS.

---

## 📌 Project Overview

This project demonstrates a production-grade DevOps platform where every code push automatically triggers a full CI/CD pipeline — building, scanning, pushing, and deploying the application to a Kubernetes cluster with zero manual intervention.

---

## 🏗️ Architecture

```
Developer (VS Code)
        │
        │ git push
        ▼
   GitHub Repo
        │
        │ Webhook trigger
        ▼
   Jenkins (EC2)
        │
        ├── Stage 1: Checkout code
        ├── Stage 2: Docker build
        ├── Stage 3: Trivy security scan
        ├── Stage 4: Push to AWS ECR
        └── Stage 5: Update deployment.yaml
                │
                │ Git commit [skip ci]
                ▼
           GitHub Repo
                │
                │ Auto detect change
                ▼
            ArgoCD
                │
                │ GitOps sync
                ▼
          AWS EKS Cluster
          ┌─────────────────────────────┐
          │  Worker Node 1 (m7i-flex)   │
          │  ┌──────────┐ ┌──────────┐  │
          │  │ App Pod  │ │ App Pod  │  │
          │  └──────────┘ └──────────┘  │
          │  ┌─────────────────────┐    │
          │  │  ArgoCD Controller  │    │
          │  └─────────────────────┘    │
          ├─────────────────────────────┤
          │  Worker Node 2 (m7i-flex)   │
          │  ┌──────────┐ ┌──────────┐  │
          │  │ App Pod  │ │ App Pod  │  │
          │  └──────────┘ └──────────┘  │
          │  ┌────────────┐ ┌────────┐  │
          │  │ Prometheus │ │Grafana │  │
          │  └────────────┘ └────────┘  │
          └─────────────────────────────┘
                │
                │ LoadBalancer
                ▼
         Public URL (ELB)
```

---

## 🛠️ Tech Stack

| Tool | Purpose |
|------|---------|
| **AWS EKS** | Managed Kubernetes cluster |
| **AWS ECR** | Private Docker image registry |
| **Jenkins** | CI/CD pipeline automation |
| **Docker** | Container build |
| **Trivy** | Container security scanning |
| **ArgoCD** | GitOps continuous deployment |
| **Helm** | Kubernetes package manager |
| **Prometheus** | Metrics collection |
| **Grafana** | Monitoring dashboards |
| **AWS IAM** | Role-based access control |

---

## 🔄 CI/CD Pipeline Flow

```
Code Push → GitHub
     ↓
Jenkins Webhook Trigger
     ↓
Stage 1: Git Checkout
     ↓
Stage 2: Docker Image Build
     ↓
Stage 3: Trivy Security Scan (HIGH/CRITICAL CVEs)
     ↓
Stage 4: Push to AWS ECR (:latest + :build_number)
     ↓
Stage 5: Update deployment.yaml [skip ci]
     ↓
ArgoCD Detects Change
     ↓
Auto Sync to EKS
     ↓
Rolling Update → App Live! ✅
```

---

## 🔐 Security Implementation

- **Trivy** scans every Docker image before deployment
- **IAM Role** attached to EC2 — no hardcoded AWS credentials
- **Least privilege principle** — ECR and EKS access only
- **Private ECR** repository for image storage
- **[skip ci]** pattern prevents infinite pipeline loops

---

## 📊 Monitoring Stack

- **Prometheus** — scrapes metrics from all pods and nodes
- **Grafana** — real-time dashboards for:
  - CPU and memory usage per node
  - Pod health and restart counts
  - Network I/O traffic
  - API server calls
- **AlertManager** — configured for alert notifications

---

## 🚀 How to Run

### Prerequisites
- AWS Account with IAM permissions
- EC2 instance (t2.micro) as control plane
- kubectl, eksctl, AWS CLI installed

### Step 1 — Create EKS Cluster
```bash
eksctl create cluster \
  --name devops-project \
  --region ap-south-1 \
  --nodegroup-name workers \
  --node-type m7i-flex.large \
  --nodes 2 \
  --nodes-min 1 \
  --nodes-max 2 \
  --managed
```

### Step 2 — Install ArgoCD
```bash
kubectl create namespace argocd
kubectl apply -n argocd -f https://raw.githubusercontent.com/argoproj/argo-cd/stable/manifests/install.yaml
kubectl patch svc argocd-server -n argocd -p '{"spec": {"type": "LoadBalancer"}}'
```

### Step 3 — Install Monitoring
```bash
helm repo add prometheus-community https://prometheus-community.github.io/helm-charts
helm repo update
helm install monitoring prometheus-community/kube-prometheus-stack \
  --namespace monitoring \
  --create-namespace
```

### Step 4 — Deploy Application
```bash
kubectl apply -f deployment.yaml
kubectl apply -f service.yaml
```

---

## 📁 Project Structure

```
todo-app/
├── index.html          # Frontend app
├── style.css           # Styling
├── Dockerfile          # Container definition
├── deployment.yaml     # Kubernetes deployment
├── service.yaml        # Kubernetes service (LoadBalancer)
└── Jenkinsfile         # CI/CD pipeline definition
```

---

## 💡 Key Learnings

- Implemented **GitOps** using ArgoCD — Git is the single source of truth
- Used **IAM Roles** instead of credentials for secure AWS access
- Solved **infinite loop** problem in CI/CD using `[skip ci]` pattern
- Configured **LoadBalancer** service for external access
- Set up **kube-prometheus-stack** for full cluster observability

---

## 🐛 Challenges & Solutions

| Challenge | Solution |
|-----------|----------|
| Jenkins pipeline looping after deployment.yaml commit | Added `[skip ci]` in commit message + SCM Skip plugin |
| EKS nodes OOMKilled with t3.micro | Upgraded to m7i-flex.large (8GB RAM) |
| ArgoCD not syncing new image | Fixed by using `:latest` tag + auto-sync policy |
| Jenkins kubectl authentication failed | Copied kubeconfig to Jenkins user home directory |
| ECR image pull error from EKS | Attached `AmazonEC2ContainerRegistryReadOnly` policy to node IAM role |

---

## 📸 Screenshots

> Add screenshots here:
> - Jenkins pipeline stages
> - ArgoCD sync dashboard  
> - Grafana monitoring dashboard
> - Live application

---

## 🔗 Related Projects

- **Project 1:** [AWS Terraform CI/CD with Docker, Jenkins, Trivy & OWASP](#)

---

## 👨‍💻 Author

**Mohit Kumar**  

---

## 📄 License

MIT License