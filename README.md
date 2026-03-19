# EKS Kubernetes Deployment With Terraform

A production-style AWS EKS cluster provisioned entirely with Terraform,
with a Dockerized application deployed using Kubernetes manifests.

## Architecture
- Custom VPC with two public subnets across two availability zones
- EKS cluster with managed node group (t3.micro)
- IAM roles for cluster and node group with least-privilege policies
- Kubernetes Deployment with liveness and readiness probes
- Kubernetes Service with LoadBalancer for external access
- Resource requests and limits on all containers
- All resources tagged and managed by Terraform

## Tools
- Terraform v1.5+
- AWS EKS
- kubectl
- Docker Hub

## Usage

### 1. Provision the cluster
cd terraform
terraform init
terraform plan
terraform apply

### 2. Configure kubectl
aws eks update-kubeconfig --region us-east-1 --name emmanuel-eks

### 3. Deploy the application
cd ../k8s
kubectl apply -f namespace.yaml
kubectl apply -f deployment.yaml
kubectl apply -f service.yaml

### 4. Verify deployment
kubectl get all -n devops-demo

### 5. Destroy when done
kubectl delete -f k8s/
cd ../terraform && terraform destroy

## Outputs
| Output              | Description                  |
|---------------------|------------------------------|
| cluster_name        | EKS cluster name             |
| cluster_endpoint    | Kubernetes API endpoint      |
| cluster_version     | Kubernetes version           |
| configure_kubectl   | kubectl config command       |

## Author
Emmanuel Ubani — Cloud & DevOps Engineer
LinkedIn: https://linkedin.com/in/ubaniemmanuel
GitHub: https://github.com/Eaglewings966