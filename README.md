AWS EKS End-to-End Project: 2048 Game Deployment with ALB Ingress
An enterprise-grade, end-to-end DevOps project demonstrating how to provision a managed Kubernetes cluster on Amazon Elastic Kubernetes Service (EKS), configure secure traffic routing using Kubernetes Ingress, and automate external load balancing via the AWS Load Balancer Controller.   
PNG

🚀 Project Overview
This project showcases a modern cloud-native architecture on AWS. It provisions an EKS cluster using eksctl, establishes secure IAM Roles for Service Accounts (IRSA), deploys the AWS Load Balancer Controller via Helm, and exposes a containerized application to the public internet using an automated AWS Application Load Balancer (ALB).   
PNG

🛠️ Tech Stack & Tools Used
Cloud Provider: Amazon Web Services (AWS) (ap-south-2)

Orchestration & Compute: Amazon EKS, eksctl, kubectl, AWS Fargate

Networking & Ingress: AWS Load Balancer Controller, Kubernetes Ingress, Target Group Bindings

Infrastructure as Code: AWS CloudFormation, IAM OIDC

Application: Containerized 2048 Game (k8s-game2048)   
PNG

📐 Architecture Visualization
[1. Infrastructure Provisioning] ➔ [2. Controller Setup (IRSA/Helm)] ➔ [3. App & Ingress Deployment] ➔ [4. External Access (ALB)]
1. Infrastructure Provisioning (eksctl): Provisions the EKS control plane, worker nodes, and networking components within a dedicated AWS VPC via automated CloudFormation templates.

2. Controller Setup (IRSA & Helm): Establishes OIDC identity provider trust, provisions the custom AWSLoadBalancerControllerIAMPolicy, and installs the controller using Helm.

3. Application & Networking (kubectl): Deploys the Kubernetes application manifests (Deployment, Service, and Ingress).

4. External Access (User Flow): The Ingress resource dynamically provisions an AWS Application Load Balancer to securely route public client traffic straight to the internal Kubernetes pods.   
PNG

📋 Step-by-Step Deployment Guide
Step 1: Create the EKS Cluster
Deploy the managed cluster with dedicated VPC and networking configurations:

Bash
eksctl create cluster --name demo-cluster --region ap-south-2 --fargate
Step 2: Set Up the AWS Load Balancer Controller
Download or create the IAM policy for the controller:

Bash
curl -o iam_policy.json https://raw.githubusercontent.com/kubernetes-sigs/aws-load-balancer-controller/v2.5.4/docs/install/iam_policy.json
aws iam create-policy --policy-name AWSLoadBalancerControllerIAMPolicy --policy-document file://iam_policy.json
Associate your OIDC provider with the cluster:

Bash
eksctl utils associate-iam-oidc-provider --cluster demo-cluster --region ap-south-2 --approve
Create an IAM service account and link it using IRSA:

Bash
eksctl create iamserviceaccount \
  --cluster=demo-cluster \
  --namespace=kube-system \
  --name=aws-load-balancer-controller \
  --role-name AmazonEKSLoadBalancerControllerRole \
  --attach-policy-arn=arn:aws:iam:::policy/AWSLoadBalancerControllerIAMPolicy \
  --approve \
  --region=ap-south-2
Install the controller using Helm:

Bash
helm repo add eks https://aws.github.io/eks-charts
helm repo update
helm upgrade -i aws-load-balancer-controller eks/aws-load-balancer-controller \
  --namespace kube-system \
  --set clusterName=demo-cluster \
  --set serviceAccount.create=false \
  --set serviceAccount.name=aws-load-balancer-controller \
  --set region=ap-south-2
Step 3: Deploy the Application & Ingress Resources
Apply the full 2048 game manifest bundle (Deployment, Service, and Ingress):

Bash
kubectl apply -f https://raw.githubusercontent.com/kubernetes-sigs/aws-load-balancer-controller/v2.5.4/docs/examples/2048/2048_full.yaml
Step 4: Verify Deployment and Access
Check the status of your ingress resource to retrieve the automatically generated external ALB DNS name:   
PNG

Bash
kubectl get ingress -n game-2048
🌐 Live Output & Validation
Once the Ingress controller provisions the load balancer, access the live game via the public AWS ALB URL:   
PNG

Access URL: k8s-game2048-ingress2-bcac0b5b37-1545777446.ap-south-2.elb.amazonaws.com

   
PNG

🧹 Resource Cleanup
To prevent ongoing cloud costs, completely remove all cluster components, security groups, VPC networks, and CloudFormation stacks:

Bash
eksctl delete cluster --name demo-cluster --region ap-south-2

