

---

## 🎯 The Project: "CloudOps Hub" — A DevOps Platform & Team Dashboard

A real-world platform where DevOps teams manage infrastructure requests, view cluster health, track deployments, and monitor costs — all while you practice every skill you listed.

### What It Does (Real Use Case)
- Team members request new environments (staging, preview, etc.)
- Admins approve via UI → Terraform provisions EKS namespace + RDS + S3 bucket automatically
- Live Kubernetes pod status, logs, and metrics displayed in real-time
- Deployment history synced from ArgoCD
- Cost tracking per environment

---

## 🏗️ Architecture Overview

```
┌─────────────────────────────────────────────────────────────────┐
│                        AWS Cloud (us-east-1)                    │
│  ┌─────────────┐    ┌─────────────┐    ┌─────────────────────┐ │
│  │   Route 53  │───▶│  CloudFront │───▶│      S3 (React)     │ │
│  └─────────────┘    └─────────────┘    │   Static Hosting    │ │
│                                        └─────────────────────┘ │
│                              │                                  │
│                         ┌────┴────┐                            │
│                         │   ALB   │                            │
│                         └────┬────┘                            │
│                              │                                  │
│                    ┌─────────┴──────────┐                      │
│                    │    EKS Cluster     │                      │
│                    │  ┌──────────────┐  │                      │
│                    │  │  Backend API │  │  Python + FastAPI    │
│                    │  │   (3 pods)   │  │  + Motor/MongoDB     │
│                    │  └──────────────┘  │                      │
│                    │  ┌──────────────┐  │                      │
│                    │  │  ArgoCD      │  │  GitOps Deployments  │
│                    │  │  (GitOps)    │  │                      │
│                    │  └──────────────┘  │                      │
│                    │  ┌──────────────┐  │                      │
│                    │  │  Monitoring  │  │  Prometheus+Grafana  │
│                    │  │  (Observab.) │  │  + Loki + AlertMgr   │
│                    │  └──────────────┘  │                      │
│                    └────────────────────┘                      │
│                              │                                  │
│                    ┌─────────┴──────────┐                      │
│                    │   DocumentDB/MongoDB │  (Managed or self-  │
│                    │   (Persistent)       │   hosted in EKS)    │
│                    └──────────────────────┘                      │
│                                                                  │
│  ┌──────────────────────────────────────────────────────────┐  │
│  │  Supporting AWS Services:                                │  │
│  │  • ECR (Docker images)  • S3 (file storage/backups)      │  │
│  │  • IAM (OIDC + IRSA)    • Secrets Manager                │  │
│  │  • Lambda (cost alerts) • SNS (notifications)            │  │
│  │  • CloudWatch (logs)    • AWS Cost Explorer API          │  │
│  └──────────────────────────────────────────────────────────┘  │
└─────────────────────────────────────────────────────────────────┘

┌─────────────────────────────────────────────────────────────────┐
│                    Local/GitHub/Terraform Cloud                 │
│  ┌─────────────┐    ┌─────────────┐    ┌─────────────────────┐ │
│  │   GitHub    │───▶│   ArgoCD    │───▶│   EKS (GitOps)      │ │
│  │   (Repos)   │    │   (Sync)    │    │   Auto-Deploy       │ │
│  └─────────────┘    └─────────────┘    └─────────────────────┘ │
│                                                                  │
│  ┌──────────────────────────────────────────────────────────┐  │
│  │  Terraform Modules (IaC):                                │  │
│  │  • networking/ (VPC, subnets, NAT, IGW)                  │  │
│  │  • eks/ (Cluster, managed nodes, IRSA, addons)           │  │
│  │  • database/ (DocumentDB or MongoDB on EC2/EKS)          │  │
│  │  • frontend/ (S3, CloudFront, Route53)                   │  │
│  │  • monitoring/ (Prometheus, Grafana via Helm)            │  │
│  │  • security/ (IAM roles, security groups, WAF)           │  │
│  └──────────────────────────────────────────────────────────┘  │
└─────────────────────────────────────────────────────────────────┘
```

---

## 📅 The 6-Week Build Plan

### Week 1: Foundation + MongoDB Deep Dive
Goal: Backend API + Database mastery

| Day | Activity | What You'll Learn |
|-----|----------|-------------------|
| 1-2 | MongoDB Fundamentals | Documents, Collections, CRUD, Indexes, Aggregation Pipeline |
| 3 | Schema Design | Embedding vs Referencing, One-to-Many, Many-to-Many for your app |
| 4-5 | Backend API (Python/FastAPI) | RESTful design, Motor async driver, Pydantic validation, error handling |
| 6-7 | Advanced MongoDB | Compound indexes, text search, transactions, change streams |

MongoDB Focus Areas for You:
- Schema Design Pattern: Design for "Environment Requests" (who requested, status, resources, timestamps)
- Aggregation Pipeline: Build a dashboard query that groups costs by team/month
- Change Streams: Real-time updates when deployment status changes
- Replication & Sharding Concepts: Even if using DocumentDB, understand the theory

Deliverable: Working API with these endpoints:
```
POST   /api/environments          # Request new env
GET    /api/environments          # List all (with filtering)
GET    /api/environments/:id      # Get details
PATCH  /api/environments/:id      # Update status (admin)
GET    /api/metrics               # Cluster metrics summary
GET    /api/deployments           # Synced from ArgoCD
POST   /api/users                 # Team member management
```

---

### Week 2: Frontend + AWS Core Infrastructure (Terraform)
Goal: React UI + AWS networking & compute

| Day | Activity | What You'll Learn |
|-----|----------|-------------------|
| 1-2 | React + Vite Setup | Modern React, React Query for API calls, Tailwind CSS |
| 3-4 | UI Components | Dashboard cards, environment request forms, status badges, charts |
| 5 | Terraform: VPC Module | 3-tier VPC (public/private subnets), NAT Gateway, VPC Endpoints |
| 6-7 | Terraform: EKS Module | EKS cluster, managed node groups, IRSA (IAM Roles for Service Accounts), OIDC provider |

Terraform Deep-Dive Tasks:
- Write reusable modules with `variables.tf`, `outputs.tf`, `main.tf`
- Use remote S3 backend with S3 Native State Locking (Terraform 1.10+) — no more DynamoDB
- Implement IRSA properly — this is critical for AWS SA skills
- Tag everything for cost allocation

Frontend Pages:
1. Dashboard — Overview cards (total envs, pending requests, monthly cost)
2. Environment Manager — Request form + table with status
3. Cluster View — Live pod status (calls K8s API via backend)
4. Deployments — GitOps sync status from ArgoCD
5. Cost Explorer — Charts showing AWS spend per environment

---

### Week 3: Kubernetes Deep Dive + Containerization
Goal: Master K8s by running your app on it

| Day | Activity | What You'll Learn |
|-----|----------|-------------------|
| 1 | Dockerize Everything | Multi-stage Dockerfiles, non-root users, distroless images |
| 2 | K8s Fundamentals | Pods, Deployments, Services, ConfigMaps, Secrets |
| 3 | Advanced K8s | HPA (Horizontal Pod Autoscaler), PDB, Resource quotas, Limits |
| 4 | K8s Networking | Services (ClusterIP, NodePort, LoadBalancer), Ingress, ALB Controller |
| 5 | K8s Security | RBAC, Network Policies, Pod Security Standards, Falco (optional) |
| 6 | Helm Charts | Templating, values.yaml, chart dependencies, Helm hooks |
| 7 | Persistence | PV/PVC, StorageClasses, StatefulSets for MongoDB |

Kubernetes Practice Tasks:
- Deploy MongoDB as a StatefulSet with persistent storage (EFS or EBS)
- Configure HPA based on CPU/memory for your backend API
- Implement Pod Disruption Budgets for zero-downtime deployments
- Create Network Policies to allow only backend → MongoDB traffic
- Set up RBAC so backend pod can only read ConfigMaps in its namespace

Helm Chart Structure:
```
helm-charts/
├── cloudops-hub/
│   ├── Chart.yaml
│   ├── values.yaml
│   ├── values-production.yaml
│   └── templates/
│       ├── deployment-backend.yaml
│       ├── deployment-frontend.yaml
│       ├── service.yaml
│       ├── ingress.yaml
│       ├── hpa.yaml
│       ├── pdb.yaml
│       ├── networkpolicy.yaml
│       └── secret.yaml
```

---

### Week 4: GitOps with ArgoCD + CI/CD Pipeline
Goal: Production-grade deployment automation

| Day | Activity | What You'll Learn |
|-----|----------|-------------------|
| 1 | ArgoCD Installation | Install via Helm, configure Git repository, understand App of Apps |
| 2 | ArgoCD Configuration | Applications, Projects, Sync policies, auto-prune, self-heal |
| 3 | Jenkins Pipeline (CI) | Multi-branch pipeline, Docker build, ECR push, Helm chart linting |
| 4 | Jenkins Pipeline (CD) | Update Helm values, Git commit back, ArgoCD auto-sync trigger |
| 5 | GitOps Workflows | PR-based deployments, preview environments, rollback strategies |
| 6 | Secrets Management | AWS Secrets Manager + External Secrets Operator, Sealed Secrets |
| 7 | Pipeline Security | Trivy scanning, Snyk, image signing with Cosign |

ArgoCD Architecture for Your Project:
```
argocd/
├── apps/
│   ├── cloudops-hub-dev.yaml      # Dev environment
│   ├── cloudops-hub-staging.yaml  # Staging
│   └── cloudops-hub-prod.yaml     # Production
└── app-of-apps.yaml               # Parent application
```

Jenkins Pipeline Stages:
```groovy
stage('Checkout')
stage('Lint & Test')
stage('Build Docker Image')
stage('Trivy Security Scan')
stage('Push to ECR')
stage('Update Helm Values')
stage('ArgoCD Sync Wait')
stage('Smoke Tests')
```

---

### Week 5: Observability + AWS Advanced Services
Goal: Production monitoring & AWS SA-level architecture

| Day | Activity | What You'll Learn |
|-----|----------|-------------------|
| 1 | Prometheus + Grafana | ServiceMonitor, custom dashboards, recording rules |
| 2 | Loki + Alertmanager | Log aggregation, log-based alerts, alert routing |
| 3 | Custom Metrics | Instrument app with Prometheus client, expose business metrics |
| 4 | AWS Cost Integration | Lambda function querying Cost Explorer API, storing in MongoDB |
| 5 | API Gateway + Lambda | Serverless components for notifications, webhooks |
| 6 | Backup & DR | Velero for EKS backups, MongoDB backup strategies |
| 7 | Performance Tuning | Uvicorn/Gunicorn worker tuning for Python, connection pooling, caching with ElastiCache |

Observability Stack (matches your resume):
- Prometheus — Metrics collection
- Grafana — Dashboards (import K8s cluster dashboard, FastAPI/Python dashboard)
- Loki — Log aggregation from all pods
- Alertmanager — PagerDuty/Slack alerts for critical issues
- Thanos (optional) — Long-term metric storage in S3

Custom Dashboards to Build:
1. Application Performance (request rate, latency, error rate)
2. Kubernetes Cluster Health (node status, pod restarts, resource usage)
3. Cost Dashboard (spend per environment, projected monthly bill)
4. MongoDB Performance (query latency, connection count, replication lag)

---

### Week 6: Security Hardening + Documentation + Polish
Goal: Production-ready + AWS SA portfolio piece

| Day | Activity | What You'll Learn |
|-----|----------|-------------------|
| 1 | Security Review | WAF on CloudFront, AWS Shield, security groups audit |
| 2 | IAM Deep Dive | Least privilege policies, permission boundaries, SCPs |
| 3 | Compliance | AWS Config rules, CloudTrail logging, VPC Flow Logs |
| 4 | Documentation | Architecture diagrams, runbooks, API documentation |
| 5 | Disaster Recovery | Test failover, backup restoration, RTO/RPO validation |
| 6 | Performance Testing | k6 load testing, K8s autoscaling validation |
| 7 | Portfolio Polish | README, demo video, blog post about the architecture |

---

## 🛠️ Tech Stack Summary

| Layer | Technology | Why |
|-------|-----------|-----|
| Frontend | React 19 + Vite + Tailwind + React Query | Modern, fast, great DX |
| Backend | Python + FastAPI + Motor | Async execution, Pydantic validation, great performance |
| Database | MongoDB 7 (DocumentDB or self-hosted) | Flexible schema, perfect for your learning |
| Containers | Docker + BuildKit + multi-stage builds | Production best practices |
| Orchestration | EKS 1.32 + Helm + Kustomize | Industry standard, your resume strength |
| GitOps | ArgoCD + GitHub | Declarative, audit-friendly deployments |
| CI/CD | Jenkins (multibranch pipeline) | Matches your professional experience |
| IaC | Terraform 1.10+ + modules + remote state | Your core skill, S3 native state locking |
| Observability | Prometheus + Grafana + Loki + Alertmanager | Matches your resume stack |
| Security | Falco + Trivy + AWS Secrets Manager | Defense in depth |
| AWS Services | EKS, ECR, ALB, S3, CloudFront, Lambda, IAM, DocumentDB | Full SA skill coverage |

---

## 📁 Repository Structure

```
cloudops-hub/
├── frontend/                    # React application
│   ├── src/
│   │   ├── components/
│   │   ├── pages/
│   │   ├── hooks/
│   │   └── services/           # API clients
│   ├── Dockerfile
│   └── nginx.conf
│
├── backend/                     # Python API (FastAPI)
│   ├── app/
│   │   ├── api/                 # API endpoints/routes
│   │   ├── core/                # Config, security, db connection
│   │   ├── models/              # Pydantic/Motor schemas
│   │   ├── services/            # Business logic
│   │   └── main.py              # FastAPI entrypoint
│   ├── Dockerfile
│   ├── requirements.txt
│   └── tests/
│
├── infrastructure/              # Terraform
│   ├── modules/
│   │   ├── networking/
│   │   ├── eks/
│   │   ├── database/
│   │   ├── frontend/
│   │   ├── monitoring/
│   │   └── security/
│   ├── environments/
│   │   ├── dev/
│   │   ├── staging/
│   │   └── production/
│   └── backend.tf              # S3 native remote state locking
│
├── kubernetes/                  # K8s manifests & Helm
│   ├── helm-charts/
│   │   └── cloudops-hub/
│   ├── base/                   # Kustomize base
│   └── overlays/
│       ├── dev/
│       ├── staging/
│       └── production/
│
├── argocd/                      # GitOps applications
│   ├── apps/
│   └── app-of-apps.yaml
│
├── jenkins/                     # CI/CD pipelines
│   ├── Jenkinsfile
│   └── scripts/
│
├── monitoring/                  # Observability configs
│   ├── prometheus/
│   ├── grafana-dashboards/
│   ├── loki/
│   └── alertmanager/
│
├── scripts/                     # Automation scripts
│   ├── setup-local.sh
│   └── cleanup-aws.sh
│
└── docs/                        # Architecture docs
    ├── architecture.md
    ├── mongodb-schema.md
    └── runbooks/
```

---

## 🚀 Getting Started (Day 1 Checklist)

1. Create GitHub repo `cloudops-hub` with the structure above
2. Set up local environment:
   - Docker Desktop with Kubernetes enabled (for local testing)
   - `kubectl`, `helm`, `terraform`, `aws-cli` installed
   - MongoDB Compass (GUI for exploring data)
3. AWS Setup:
   - Create AWS account (or use existing)
   - Set up Terraform remote state S3 bucket with native state locking
   - Configure AWS CLI with least-privilege IAM user
4. Start MongoDB locally:
   ```bash
   docker run -d -p 27017:27017 --name mongo mongo:7
   ```
5. Initialize backend project:
   ```bash
   cd backend && python -m venv venv
   # Activate venv: .\venv\Scripts\activate (Windows) or source venv/bin/activate (Unix)
   pip install fastapi uvicorn motor pydantic-settings dnspython
   pip freeze > requirements.txt
   ```

---

## 💡 Pro Tips for Your Learning

1. Document Everything — Write a blog post per week; this becomes interview material
2. Break Things Intentionally — Delete a pod, fill up disk, kill a node — learn recovery
3. Cost Alerts — Set AWS budget alerts at $50/week to avoid surprises
4. Use `kind` or `k3d` locally before spinning up EKS to save money
5. Join communities — Kubernetes Slack, AWS Discord, r/devops for help

---

## 🏆 Portfolio Outcome

After 6 weeks, you'll have:
- ✅ A production-grade full-stack app on your GitHub
- ✅ Terraform modules reusable for future projects
- ✅ ArgoCD GitOps workflow you can demo in interviews
- ✅ MongoDB expertise with complex aggregations and indexing
- ✅ K8s deep knowledge — HPA, networking, security, Helm
- ✅ Observability stack — custom Grafana dashboards, alerting
- ✅ AWS SA-level architecture — multi-AZ, auto-scaling, cost-optimized

This project directly extends your resume experience (EKS, Jenkins, Terraform, ArgoCD, MongoDB) and gives you stories to tell in interviews about problems you solved.

Want me to dive deeper into any specific week, or shall we start with the MongoDB schema design and backend API structure?