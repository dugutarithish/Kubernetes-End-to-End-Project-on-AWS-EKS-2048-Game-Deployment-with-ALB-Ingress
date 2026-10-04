# 🚀 AWS EKS 2048 Game Deployment with ALB Ingress

> End-to-end Kubernetes and AWS DevOps project demonstrating containerized application deployment on Amazon EKS with AWS Load Balancer Controller, IAM Roles for Service Accounts (IRSA), Helm, and Application Load Balancer (ALB).

---

## 📌 Project Overview

This project demonstrates how to deploy a containerized **2048 web application** on **Amazon Elastic Kubernetes Service (EKS)** and expose it to the internet using an AWS Application Load Balancer.

The infrastructure and application deployment are implemented using:

- Amazon EKS
- AWS Fargate
- Kubernetes
- eksctl
- Helm
- AWS Load Balancer Controller
- IAM OIDC
- IRSA
- Kubernetes Ingress
- AWS Application Load Balancer
- AWS CloudFormation

The project follows a complete flow from **EKS cluster creation → IAM configuration → Load Balancer Controller → Kubernetes deployment → ALB provisioning → application access**.

---

## 🏗️ Architecture

```text
<img width="1024" height="520" alt="eks project" src="https://github.com/user-attachments/assets/003d2bb7-2330-4f33-87ec-3e484092dedc" />


| Category | Technologies |
|---|---|
| Cloud | AWS |
| Container Orchestration | Amazon EKS |
| Compute | AWS Fargate |
| Kubernetes CLI | kubectl |
| Cluster Management | eksctl |
| Package Manager | Helm |
| Load Balancing | AWS Application Load Balancer |
| Ingress | Kubernetes Ingress |
| AWS Integration | AWS Load Balancer Controller |
| IAM | IAM, OIDC, IRSA |
| Infrastructure | AWS CloudFormation |
| Application | 2048 Game |



📋 Deployment Process
1️⃣ Create the EKS Cluster
Create an EKS cluster using eksctl.

eksctl create cluster \
  --name demo-cluster \
  --region ap-south-2 \
  --fargate
This provisions:
- Amazon EKS control plane
- AWS Fargate compute
- VPC networking
- Subnets
- Security groups
- Required CloudFormation resources

Verify the cluster:
aws eks update-kubeconfig \
  --name demo-cluster \
  --region ap-south-2

kubectl get nodes

kubectl get pods -A


2️⃣ Configure AWS Load Balancer Controller
The AWS Load Balancer Controller allows Kubernetes Ingress resources to automatically create and manage AWS Application Load Balancers.
Download the IAM Policy

curl -o iam_policy.json \
https://raw.githubusercontent.com/kubernetes-sigs/aws-load-balancer-controller/v2.5.4/docs/install/iam_policy.json

Create the IAM policy:
aws iam create-policy \
  --policy-name AWSLoadBalancerControllerIAMPolicy \
  --policy-document file://iam_policy.json
Replace <AWS_ACCOUNT_ID> with your AWS account ID in the IAM policy ARN used below.


3️⃣ Configure IAM OIDC Provider
Associate the EKS cluster with an IAM OIDC provider.
eksctl utils associate-iam-oidc-provider \
  --cluster demo-cluster \
  --region ap-south-2 \
  --approve

  Verify:
  aws eks describe-cluster \
  --name demo-cluster \
  --region ap-south-2 \
  --query "cluster.identity.oidc.issuer"

  4️⃣ Create IAM Service Account
Create a Kubernetes service account and associate it with an IAM role using IRSA.
eksctl create iamserviceaccount \
  --cluster=demo-cluster \
  --namespace=kube-system \
  --name=aws-load-balancer-controller \
  --role-name=AmazonEKSLoadBalancerControllerRole \
  --attach-policy-arn=arn:aws:iam::<AWS_ACCOUNT_ID>:policy/AWSLoadBalancerControllerIAMPolicy \
  --approve \
  --region=ap-south-2

  Why IRSA?
IRSA allows Kubernetes workloads to securely access AWS services without storing AWS access keys inside containers.
Kubernetes Service Account
          │
          ▼
       OIDC
          │
          ▼
       IAM Role
          │
          ▼
    AWS Permissions

    5️⃣ Install AWS Load Balancer Controller
Add the AWS EKS Helm repository:

helm repo add eks https://aws.github.io/eks-charts

Update Helm repositories:
helm repo update

Install the controller:

helm upgrade --install aws-load-balancer-controller \
  eks/aws-load-balancer-controller \
  --namespace kube-system \
  --set clusterName=demo-cluster \
  --set serviceAccount.create=false \
  --set serviceAccount.name=aws-load-balancer-controller \
  --set region=ap-south-2

  Verify the controller:
  kubectl get deployment \
  -n kube-system \
  aws-load-balancer-controller

  Check the pods:
  kubectl get pods -n kube-system


  6️⃣ Deploy the 2048 Application
Deploy the Kubernetes resources containing:
- Namespace
- Deployment
- Service
- Ingress
kubectl apply -f \
https://raw.githubusercontent.com/kubernetes-sigs/aws-load-balancer-controller/v2.5.4/docs/examples/2048/2048_full.yaml

Verify the application:
kubectl get pods -n game-2048
kubectl get service -n game-2048
kubectl get deployment -n game-2048

7️⃣ Verify the Ingress
Check the Kubernetes Ingress:
kubectl get ingress -n game-2048

Example:
NAME                  CLASS   HOSTS   ADDRESS
ingress-2048          alb     *       xxx.ap-south-2.elb.amazonaws.com

🌐 Application Access
After the ALB has been provisioned, retrieve the ALB DNS name:kubectl get pods -n game-2048

kubectl get ingress -n game-2048

Open the generated ALB address in your browser.
Example:
http://<ALB-DNS-NAME>
http://<ALB-DNS-NAME>

🔍 Verification
Use the following commands to verify the complete deployment.
Kubernetes Resources

kubectl get pods -n game-2048
kubectl get svc -n game-2048
kubectl get ingress -n game-2048

Load Balancer Controller

kubectl get pods -n kube-system \
  -l app.kubernetes.io/name=aws-load-balancer-controller

Detailed Ingress Information
  kubectl describe ingress -n game-2048

Check All Resources
kubectl get all -n game-2048

