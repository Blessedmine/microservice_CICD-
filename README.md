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
eksctl create cluster --name microservice --region us-east-1

To enable and use AWS IAM roles for Kubernetes service accounts on our EKS cluster, we must create & associate OIDC identity provider.
To do so using eksctl we can use the below command.

eksctl utils associate-iam-oidc-provider \
    --region us-east-1 \
    --cluster microservice \
    --approve
    
These add-ons will create the respective IAM policies for us automatically within our Node Group role.

# Create Public Node Group   
eksctl create nodegroup --cluster=microservice \
                       --region=us-east-1 \
                       --name=mg-demo \
                       --node-type=t3.medium \
                       --nodes=2 \
                       --nodes-min=2 \
                       --nodes-max=4 \
                       --node-volume-size=20 \
                       --ssh-access \
                       --ssh-public-key=pipeline \
                       --managed \
                       --asg-access \
                       --external-dns-access \
                       --full-ecr-access \
                       --appmesh-access \
                       --alb-ingress-access

After the nodes are ready Do:
  kubectl config get-contexts
 To know the current context Run:
  kubectl get nodes
  kubectl get pods -n kube-system
  kubectl top pods -n kube-system
    
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
kubectl apply -f microservice_CICD/frontend/frontend-deployment.yaml

### **2.2 Service Deployment**
#### **Steps:**
1. **Create Kubernetes Manifests for the microservices**
   ```
2. **Deploy Services**
   ```sh
   kubectl apply -f kubernetes/manifests/frontend-deployment.yaml
   ```  

3. **Configure Ingress**
- Install **NGINX Ingress Controller**:
  ```sh
  helm install ingress-nginx ingress-nginx/ingress-nginx
  ```  
- Define `ingress.yaml` for routing.

# Add the Helm repository (if not already added)
helm repo add ingress-nginx https://kubernetes.github.io/ingress-nginx
helm repo update

kubectl apply -f ingress.yaml

# Install NGINX Ingress Controller in a dedicated namespace
helm upgrade --install ingress-nginx ingress-nginx/ingress-nginx --namespace ingress-nginx --create-namespace --set controller.service.type=LoadBalancer

Check if Helm release exists

helm list -A

kubectl get deployments -n ingress-nginx
kubectl get pods -n ingress-nginx
kubectl get ingress -A
kubectl get svc -n ingress-nginx

### **3.1 Scaling & Secrets**
#### **Steps:**
1. **Horizontal Pod Autoscaler (HPA)**

2. **Secrets & ConfigMaps**
- Store secrets:
  ```sh
  kubectl create secret generic db-secret --from-literal=password=1234
  ```  
- Use ConfigMaps for environment variables.

**Deploy Using ArgoCD**
- Install ArgoCD:
  ```sh
  kubectl create namespace argocd
  kubectl apply -n argocd -f https://raw.githubusercontent.com/argoproj/argo-cd/stable/manifests/install.yaml
  ```  
  LoadBalancer (if using AWS EKS)
  kubectl patch svc argocd-server -n argocd -p '{"spec": {"type": "LoadBalancer"}}'

This gives you a public ELB DNS name in AWS.

Get the Admin Password

The initial username is admin.
Get the auto-generated password:

kubectl -n argocd get secret argocd-initial-admin-secret -o jsonpath="{.data.password}" | base64 -d; echo

Login to ArgoCD CLI

Install argocd CLI (from ArgoCD releases
), then:

argocd login <ARGOCD_SERVER>


Example (if port-forwarding):

argocd login localhost:8080

Create app.yaml and Apply it:

kubectl apply -f app.yaml -n argocd

ArgoCD will now sync your cluster with the Git repo.

Observability & Cleanup
Phase 4: Observability & Cleanup
Install Prometheus & Grafana
helm repo add prometheus-community https://prometheus-community.github.io/helm-charts
helm repo update
kubectl create namespace monitoring
helm install monitoring prometheus-community/kube-prometheus-stack -n monitoring

Access Grafana
kubectl get svc -n monitoring
kubectl port-forward svc/monitoring-grafana -n monitoring 3000:80


Default login:

Username: admin

Password:

kubectl get secret monitoring-grafana -n monitoring -o jsonpath="{.data.admin-password}" | base64 --decode ; echo


Import dashboards for CPU, memory, and pod metrics.

References

https://docs.aws.amazon.com/eks/latest/userguide/what-is-eks.html

https://argo-cd.readthedocs.io/

https://github.com/prometheus-community/helm-charts
