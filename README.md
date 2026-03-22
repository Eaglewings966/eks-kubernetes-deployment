# EKS Kubernetes Deployment With Terraform and Fargate

![AWS](https://img.shields.io/badge/AWS-EKS-FF9900?style=for-the-badge&logo=amazonaws&logoColor=white)
![Terraform](https://img.shields.io/badge/Terraform-1.5+-7B42BC?style=for-the-badge&logo=terraform&logoColor=white)
![Kubernetes](https://img.shields.io/badge/Kubernetes-1.29-326CE5?style=for-the-badge&logo=kubernetes&logoColor=white)
![Docker](https://img.shields.io/badge/Docker-Hub-2496ED?style=for-the-badge&logo=docker&logoColor=white)
![eksctl](https://img.shields.io/badge/eksctl-0.220.0-FF9900?style=for-the-badge&logo=amazonaws&logoColor=white)
![License](https://img.shields.io/badge/License-MIT-green?style=for-the-badge)

A production-grade AWS EKS deployment using Terraform for infrastructure and AWS Fargate for serverless container compute — demonstrating real-world cloud infrastructure skills including troubleshooting AWS account restrictions and implementing alternative deployment strategies.

---

## What This Project Demonstrates

- Provisioning a production-grade EKS cluster using both Terraform and eksctl
- AWS Fargate for serverless container compute — zero EC2 instances to manage
- Fargate profiles targeting specific namespaces for workload isolation
- Kubernetes Deployment with liveness and readiness probes
- Real-world AWS account troubleshooting — diagnosing and resolving EC2 Fleet Request and vCPU quota restrictions
- Multiple deployment strategies — managed node groups, self-managed ASG, and Fargate
- Full infrastructure teardown to avoid unnecessary AWS charges

---

## Tech Stack

| Tool | Purpose |
|------|---------|
| Terraform v1.5+ | Infrastructure as Code — VPC, IAM roles, security groups |
| eksctl v0.220.0 | EKS cluster provisioning with Fargate support |
| AWS EKS | Managed Kubernetes control plane |
| AWS Fargate | Serverless compute for containers — no EC2 needed |
| kubectl | Kubernetes CLI for deploying and managing workloads |
| Docker Hub | Container registry hosting the application image |
| AWS IAM | Least-privilege roles for EKS cluster and node group |

---

## Project Structure

```
eks-kubernetes-deployment/
├── terraform/
│   ├── main.tf          # VPC, IAM roles, security groups, launch template, ASG
│   ├── variables.tf     # Input variables including capacity type and AMI ID
│   ├── outputs.tf       # Cluster endpoint, name, kubectl command
│   ├── versions.tf      # Terraform and provider version constraints
│   └── terraform.tfvars # Variable values (gitignored — never committed)
├── k8s/
│   ├── namespace.yaml   # devops-demo namespace
│   ├── deployment.yaml  # App deployment with probes and resource limits
│   └── service.yaml     # ClusterIP service on port 80
├── .gitignore
└── README.md
```

---

## Account Requirements

This project requires the following AWS account quotas in us-east-1:

- Running On-Demand Standard vCPUs: minimum 4
- EC2 Fleet Requests: minimum 5

New AWS accounts may have restricted quotas. Check your limits before provisioning:

```bash
aws service-quotas get-service-quota --service-code ec2 --quota-code L-1216C47A --region us-east-1
```

If you encounter quota restrictions, use the Fargate deployment path below — it requires zero EC2 instances and bypasses all vCPU and Fleet Request limits.

---

## Deployment — Fargate Path (Recommended for New Accounts)

### 1. Prerequisites

- AWS CLI configured
- eksctl installed
- kubectl installed

### 2. Create the EKS Fargate cluster

```bash
eksctl create cluster --name emmanuel-eks --region us-east-1 --fargate
```

This takes 15-20 minutes. eksctl creates the VPC, subnets, IAM roles, and Fargate profiles automatically.

### 3. Add Fargate profile for your app namespace

```bash
eksctl create fargateprofile --cluster emmanuel-eks --region us-east-1 --name fp-devops-demo --namespace devops-demo
```

### 4. Deploy the application

```bash
kubectl apply -f k8s/namespace.yaml
kubectl apply -f k8s/deployment.yaml
kubectl apply -f k8s/service.yaml
```

### 5. Verify deployment

```bash
kubectl get nodes
kubectl get pods -n devops-demo
kubectl get svc -n devops-demo
```

### 6. Access the application

```bash
kubectl port-forward -n devops-demo deployment/devops-demo-app 8080:3000
```

Open your browser at http://localhost:8080

### 7. Destroy when done

```bash
eksctl delete cluster --name emmanuel-eks --region us-east-1
```

---

## Deployment — Terraform Path (Standard AWS Accounts)

### 1. Configure your variables

Create a terraform.tfvars file inside the terraform/ folder:

```hcl
aws_region         = "us-east-1"
project_name       = "emmanuel-eks"
cluster_version    = "1.29"
node_instance_type = "t3.medium"
node_ami_id        = "ami-0a23644f1ead7eb05"
desired_nodes      = 1
min_nodes          = 1
max_nodes          = 3
capacity_type      = "SPOT"
```

### 2. Provision the infrastructure

```bash
cd terraform
terraform init
terraform plan
terraform apply
```

### 3. Configure kubectl

```bash
aws eks update-kubeconfig --region us-east-1 --name emmanuel-eks
```

### 4. Deploy the application

```bash
cd ../k8s
kubectl apply -f namespace.yaml
kubectl apply -f deployment.yaml
kubectl apply -f service.yaml
```

### 5. Destroy when done

```bash
kubectl delete namespace devops-demo
cd ../terraform && terraform destroy
```

---

## Key Lessons Learned

**Check your EC2 vCPU quota before writing any Terraform.** New AWS accounts can have limits as low as 1 vCPU. A t3.medium needs 2. Run the quota check command before you start and save yourself days of debugging.

**EKS managed node groups use EC2 Fleet internally.** Even with a launch template or Spot capacity type, EKS routes node group requests through EC2 Fleet. If your account has a low Fleet Request quota, every node group attempt will fail regardless of instance type or capacity type.

**Self-managed node groups bypass EC2 Fleet.** Using an AWS Auto Scaling Group directly with a launch template avoids the Fleet Request quota entirely — but still requires sufficient vCPU quota.

**Fargate bypasses everything.** AWS Fargate requires zero EC2 instances, zero vCPU quota, and zero Fleet Request quota. It is the cleanest solution for new AWS accounts with restricted limits.

**Fargate requires private subnets.** Fargate profiles cannot use public subnets. Always let eksctl create the VPC automatically — it provisions both public and private subnets correctly.

**Classic Load Balancers cannot reach Fargate pods.** Fargate pods live in private subnets. Classic LBs cannot register them as targets. Use port-forward for local access or deploy the AWS Load Balancer Controller for production ALB support.

**Never commit terraform.tfstate to GitHub.** Your state file contains AWS account IDs, resource IDs, and infrastructure details. Always verify your .gitignore covers .terraform/, terraform.tfstate, terraform.tfstate.backup, and .tfvars before pushing.

**Never store project files in C:/Windows/System32.** Git cannot write config files in protected Windows directories. Always create projects in C:/Projects or your user home directory.

---

## Author

**Emmanuel Ubani** — Cloud & DevOps Engineer

[![LinkedIn](https://img.shields.io/badge/LinkedIn-Connect-0A66C2?style=flat&logo=linkedin)](https://linkedin.com/in/ubaniemmanuel)
[![GitHub](https://img.shields.io/badge/GitHub-Eaglewings966-181717?style=flat&logo=github)](https://github.com/Eaglewings966)
[![Hashnode](https://img.shields.io/badge/Hashnode-Blog-2962FF?style=flat&logo=hashnode)](https://emmanuelubani.hashnode.dev)
[![Medium](https://img.shields.io/badge/Medium-Articles-000000?style=flat&logo=medium)](https://medium.com/@emmaubani966)

---

*Part of my Cloud and DevOps engineering portfolio. Built with real AWS infrastructure, real quota restrictions, and real problem solving.*
