**Project Documentation: Microservices on Kubernetes
*Overview
This project involves building, containerizing, and deploying 3 microservices on Kubernetes (EKS). The implementation is divided into four phases:
Foundation Setup – Environment preparation and containerization.
Kubernetes Deployment – Cluster setup and service deployment.
Production Features – Scaling, secrets, and Helm charts.
Observability & Service Mesh – Monitoring with Prometheus/Grafana and final documentation.

**Phase 1: Foundation Setup
*1.1 Environment Preparation
**Steps:
1. Set up a Cloud Account
Create an account on AWS (for EKS).
Enable Kubernetes service (EKS) and container registry (ECR).

2. Install Local Tools
Install Docker (for containerization).
Install kubectl (Kubernetes CLI).
Install Helm (for package management).
Install IntelliJ.

3. Set Up GitHub Repository.  

1.2 Containerization
Steps:
1. Build Microservices
2. Create Dockerfiles 
3. Build and Push Images
Build images:
docker build -t <registry>/<service-name>:<tag> .

Create ecr
aws ecr create-repository --repository-name service-name
Push to AWS ECR:
docker push <registry>/<service-name>:<tag>

You can also follow the command line on the aws if you are using AWS


---

## **Phase 2: Kubernetes Deployment**

### **2.1 Cluster Setup**
#### **Steps:**
1. **Provision AKS/EKS Cluster**
- **AWS EKS**:
```sh
eksctl create cluster --name my-cluster --region us-east-1

Configure kubectl Access
For EKS:
 aws eks update-kubeconfig --name <cluster_name> --region <region> 

Set Up Namespaces & RBAC
Create namespaces:
kubectl create namespace frontend
kubectl create namespace product
kubectl create namespace shopping-cart

Apply RBAC policies if needed.

2.2 Service Deployment
Steps:
1. Create Kubernetes Manifests
2. Deploy Service