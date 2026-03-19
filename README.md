# EKS Kubernetes Deployment With Terraform

![AWS](https://img.shields.io/badge/AWS-EKS-FF9900?style=for-the-badge&logo=amazonaws&logoColor=white)
![Terraform](https://img.shields.io/badge/Terraform-1.5+-7B42BC?style=for-the-badge&logo=terraform&logoColor=white)
![Kubernetes](https://img.shields.io/badge/Kubernetes-1.29-326CE5?style=for-the-badge&logo=kubernetes&logoColor=white)
![Docker](https://img.shields.io/badge/Docker-Hub-2496ED?style=for-the-badge&logo=docker&logoColor=white)
![License](https://img.shields.io/badge/License-MIT-green?style=for-the-badge)

A production-grade AWS EKS cluster provisioned entirely with Terraform, with a Dockerized Node.js application deployed using Kubernetes manifests — demonstrating real-world cloud infrastructure skills.

---

## Architecture

```
                        ┌─────────────────────────────────────────────┐
                        │               AWS Cloud (us-east-1)          │
                        │                                               │
                        │   ┌─────────────────────────────────────┐   │
                        │   │            Custom VPC                │   │
                        │   │                                      │   │
                        │   │  ┌──────────────┐ ┌──────────────┐  │   │
                        │   │  │  Public Sub  │ │  Public Sub  │  │   │
                        │   │  │  us-east-1a  │ │  us-east-1b  │  │   │
                        │   │  └──────┬───────┘ └───────┬──────┘  │   │
                        │   │         │                  │         │   │
                        │   │  ┌──────▼──────────────────▼──────┐  │   │
                        │   │  │         EKS Cluster             │  │   │
                        │   │  │       (emmanuel-eks)            │  │   │
                        │   │  │                                 │  │   │
                        │   │  │  ┌───────────────────────────┐  │  │   │
                        │   │  │  │     Managed Node Group    │  │  │   │
                        │   │  │  │   t3.medium (1-2 nodes)   │  │  │   │
                        │   │  │  │                           │  │  │   │
                        │   │  │  │  ┌─────────────────────┐  │  │  │   │
                        │   │  │  │  │   Namespace:        │  │  │  │   │
                        │   │  │  │  │   devops-demo        │  │  │  │   │
                        │   │  │  │  │                     │  │  │  │   │
                        │   │  │  │  │  ┌───────────────┐  │  │  │  │   │
                        │   │  │  │  │  │  Deployment   │  │  │  │  │   │
                        │   │  │  │  │  │  (Node.js app)│  │  │  │  │   │
                        │   │  │  │  │  │  port: 3000   │  │  │  │  │   │
                        │   │  │  │  │  └───────┬───────┘  │  │  │  │   │
                        │   │  │  │  │          │          │  │  │  │   │
                        │   │  │  │  │  ┌───────▼───────┐  │  │  │  │   │
                        │   │  │  │  │  │   Service     │  │  │  │  │   │
                        │   │  │  │  │  │ LoadBalancer  │  │  │  │  │   │
                        │   │  │  │  │  │   port: 80    │  │  │  │  │   │
                        │   │  │  │  │  └───────┬───────┘  │  │  │  │   │
                        │   │  │  │  └──────────┼──────────┘  │  │  │   │
                        │   │  │  └─────────────┼─────────────┘  │  │   │
                        │   │  └─────────────────┼────────────────┘  │   │
                        │   └─────────────────────┼──────────────────┘   │
                        └─────────────────────────┼────────────────────  ┘
                                                   │
                                            ┌──────▼──────┐
                                            │   Internet  │
                                            │   (port 80) │
                                            └─────────────┘
```

---

## What This Project Demonstrates

- Provisioning a production-grade EKS cluster using Terraform with zero manual console clicks
- IAM roles and policies following the principle of least privilege for both cluster and node group
- Kubernetes Deployment with liveness and readiness probes, resource requests and limits
- LoadBalancer Service exposing the application externally on port 80
- Namespace isolation for clean workload separation
- Full infrastructure teardown with `terraform destroy` to avoid unnecessary AWS charges

---

## Tech Stack

| Tool | Purpose |
|------|---------|
| Terraform v1.5+ | Infrastructure as Code — provisions all AWS resources |
| AWS EKS | Managed Kubernetes control plane |
| AWS EC2 (t3.medium) | Worker nodes running the application pods |
| kubectl | Kubernetes CLI for deploying and managing workloads |
| Docker Hub | Container registry hosting the application image |
| AWS IAM | Least-privilege roles for EKS cluster and node group |

---

## Project Structure

```
eks-kubernetes-deployment/
├── terraform/
│   ├── main.tf          # EKS cluster, node group, IAM roles, VPC config
│   ├── variables.tf     # Input variables
│   ├── outputs.tf       # Cluster endpoint, name, kubectl command
│   ├── versions.tf      # Terraform and provider version constraints
│   └── terraform.tfvars # Variable values (gitignored — not committed)
├── k8s/
│   ├── namespace.yaml   # devops-demo namespace
│   ├── deployment.yaml  # App deployment with probes and resource limits
│   └── service.yaml     # LoadBalancer service on port 80
├── .gitignore
└── README.md
```

---

## Prerequisites

- AWS CLI configured with appropriate permissions
- Terraform v1.5+ installed
- kubectl installed
- AWS account with sufficient EC2 vCPU quota for t3.medium instances

---

## Usage

### 1. Clone the repository

```bash
git clone https://github.com/Eaglewings966/eks-kubernetes-deployment.git
cd eks-kubernetes-deployment
```

### 2. Configure your variables

Create a `terraform.tfvars` file inside the `terraform/` folder:

```hcl
region       = "us-east-1"
cluster_name = "emmanuel-eks"
```

### 3. Provision the cluster

```bash
cd terraform
terraform init
terraform plan
terraform apply
```

> Cluster creation takes approximately 12-18 minutes.

### 4. Configure kubectl

```bash
aws eks update-kubeconfig --region us-east-1 --name emmanuel-eks
```

### 5. Deploy the application

```bash
cd ../k8s
kubectl apply -f namespace.yaml
kubectl apply -f deployment.yaml
kubectl apply -f service.yaml
```

### 6. Verify deployment

```bash
kubectl get nodes
kubectl get all -n devops-demo
```

### 7. Get the LoadBalancer URL

```bash
kubectl get svc -n devops-demo
```

Copy the `EXTERNAL-IP` value and open it in your browser on port 80.

### 8. Destroy when done

```bash
kubectl delete namespace devops-demo
cd ../terraform && terraform destroy
```

> Always delete the namespace before running terraform destroy — this ensures the AWS Load Balancer is removed cleanly before the VPC is destroyed.

---

## Outputs

| Output | Description |
|--------|-------------|
| cluster_name | EKS cluster name |
| cluster_endpoint | Kubernetes API server endpoint |
| cluster_version | Kubernetes version running on the cluster |
| configure_kubectl | Ready-to-run kubectl config command |

---

## Key Lessons Learned

- New AWS accounts have restricted EC2 vCPU quotas (as low as 1 vCPU) — always check your Service Quotas before provisioning EKS
- EKS managed node groups use EC2 Fleet requests internally — t3.medium is the minimum recommended instance type for EKS worker nodes
- Always delete Kubernetes LoadBalancer services before running terraform destroy to avoid orphaned AWS ELB resources
- Terraform state files contain sensitive infrastructure details — never commit them to version control

---

## Author

**Emmanuel Ubani** — Cloud & DevOps Engineer

[![LinkedIn](https://img.shields.io/badge/LinkedIn-Connect-0A66C2?style=flat&logo=linkedin)](https://linkedin.com/in/ubaniemmanuel)
[![GitHub](https://img.shields.io/badge/GitHub-Eaglewings966-181717?style=flat&logo=github)](https://github.com/Eaglewings966)
[![Hashnode](https://img.shields.io/badge/Hashnode-Blog-2962FF?style=flat&logo=hashnode)](https://emmanuelubani.hashnode.dev)
[![Medium](https://img.shields.io/badge/Medium-Articles-000000?style=flat&logo=medium)](https://medium.com/@eaglewings966)

---

*Part of my Cloud & DevOps engineering portfolio. Built with real AWS infrastructure, not simulators.*