# 🚀 AWS EKS 2048 Game Deployment with ALB Ingress

> **End-to-end Kubernetes and AWS DevOps project** demonstrating containerized application deployment on Amazon EKS using AWS Fargate, Helm, IRSA, AWS Load Balancer Controller, Kubernetes Ingress, and Application Load Balancer (ALB).

> [!IMPORTANT]
> **AWS Region:** `ap-south-2`  
> **Kubernetes Platform:** Amazon EKS  
> **Compute:** AWS Fargate

---

## 📌 Project Overview

This project demonstrates how to deploy a containerized **2048 web application** on **Amazon Elastic Kubernetes Service (EKS)** and expose it to the internet using an **AWS Application Load Balancer (ALB)**.

The project covers the complete deployment lifecycle:

```text
EKS Cluster
     ↓
IAM / OIDC
     ↓
IRSA
     ↓
AWS Load Balancer Controller
     ↓
Kubernetes Application
     ↓
Ingress
     ↓
AWS Application Load Balancer
     ↓
Public Application
```

### 🎯 Project Objectives

- Provision an **Amazon EKS cluster**
- Run workloads using **AWS Fargate**
- Configure **IAM OIDC**
- Implement **IAM Roles for Service Accounts (IRSA)**
- Install **AWS Load Balancer Controller**
- Deploy a containerized **2048 application**
- Configure **Kubernetes Ingress**
- Automatically provision an **AWS Application Load Balancer**
- Expose the application through a public AWS endpoint
- Verify and troubleshoot Kubernetes resources

---

# 🏗️ Architecture

<img width="1024" height="520" alt="AWS EKS Architecture" src="https://github.com/user-attachments/assets/003d2bb7-2330-4f33-87ec-3e484092dedc" />

### 🔄 Request Flow

```text
                         ┌─────────────────────┐
                         │       Internet      │
                         │        User         │
                         └──────────┬──────────┘
                                    │
                                    ▼
                    ┌─────────────────────────────┐
                    │ AWS Application Load        │
                    │ Balancer (ALB)              │
                    └─────────────┬───────────────┘
                                  │
                                  ▼
                    ┌─────────────────────────────┐
                    │ Kubernetes Ingress          │
                    └─────────────┬───────────────┘
                                  │
                                  ▼
                    ┌─────────────────────────────┐
                    │ AWS Load Balancer           │
                    │ Controller                  │
                    └─────────────┬───────────────┘
                                  │
                                  ▼
                    ┌─────────────────────────────┐
                    │ Kubernetes Service          │
                    │       2048 Game             │
                    └─────────────┬───────────────┘
                                  │
                       ┌──────────┴──────────┐
                       ▼                     ▼
                ┌──────────────┐      ┌──────────────┐
                │  2048 Pod    │      │  2048 Pod    │
                │  Fargate     │      │  Fargate     │
                └──────────────┘      └──────────────┘
```

---

# 🛠️ Technology Stack

| Category | Technology |
|---|---|
| ☁️ Cloud Provider | **AWS** |
| ☸️ Container Orchestration | **Amazon EKS** |
| ⚙️ Compute | **AWS Fargate** |
| 🖥️ Kubernetes CLI | **kubectl** |
| 🚀 Cluster Management | **eksctl** |
| 📦 Package Manager | **Helm** |
| 🌐 Load Balancer | **AWS Application Load Balancer** |
| 🔀 Ingress | **Kubernetes Ingress** |
| ⚡ AWS Integration | **AWS Load Balancer Controller** |
| 🔐 IAM | **IAM, OIDC, IRSA** |
| 🏗️ Infrastructure | **AWS CloudFormation** |
| 🎮 Application | **2048 Game** |

---

# 📋 Deployment Process

## 1️⃣ Create the EKS Cluster

Create the EKS cluster using `eksctl`.

```bash
eksctl create cluster \
  --name demo-cluster \
  --region ap-south-2 \
  --fargate
```

### What this creates

- **Amazon EKS control plane**
- **AWS Fargate compute**
- **VPC networking**
- **Subnets**
- **Security groups**
- **CloudFormation resources**

> [!NOTE]
> `eksctl` automatically provisions several AWS resources required for the EKS cluster.

### Configure kubectl

```bash
aws eks update-kubeconfig \
  --name demo-cluster \
  --region ap-south-2
```

### Verify the cluster

```bash
kubectl get nodes
```

```bash
kubectl get pods -A
```

> [!TIP]
> Run `kubectl get pods -A` to verify that the Kubernetes system components are running correctly.

---

# 2️⃣ Configure AWS Load Balancer Controller

The **AWS Load Balancer Controller** allows Kubernetes resources to automatically create and manage AWS load balancers.

### Download the IAM Policy

```bash
curl -o iam_policy.json \
https://raw.githubusercontent.com/kubernetes-sigs/aws-load-balancer-controller/v2.5.4/docs/install/iam_policy.json
```

### Create the IAM Policy

```bash
aws iam create-policy \
  --policy-name AWSLoadBalancerControllerIAMPolicy \
  --policy-document file://iam_policy.json
```

> [!IMPORTANT]
> The IAM policy ARN must contain your **AWS Account ID**.

---

# 3️⃣ Configure IAM OIDC Provider

Associate the EKS cluster with an **IAM OIDC provider**.

```bash
eksctl utils associate-iam-oidc-provider \
  --cluster demo-cluster \
  --region ap-south-2 \
  --approve
```

### Verify OIDC

```bash
aws eks describe-cluster \
  --name demo-cluster \
  --region ap-south-2 \
  --query "cluster.identity.oidc.issuer"
```

A successful response should return an OIDC issuer URL.

---

# 4️⃣ Create IAM Service Account with IRSA

Create a Kubernetes service account and associate it with an IAM role.

```bash
eksctl create iamserviceaccount \
  --cluster=demo-cluster \
  --namespace=kube-system \
  --name=aws-load-balancer-controller \
  --role-name=AmazonEKSLoadBalancerControllerRole \
  --attach-policy-arn=arn:aws:iam::<AWS_ACCOUNT_ID>:policy/AWSLoadBalancerControllerIAMPolicy \
  --approve \
  --region=ap-south-2
```

> [!WARNING]
> Replace `<AWS_ACCOUNT_ID>` with your actual AWS account ID before running the command.

### 🔐 Why IRSA?

**IAM Roles for Service Accounts (IRSA)** allows Kubernetes workloads to securely access AWS services without storing AWS access keys inside containers.

```text
Kubernetes Service Account
            │
            ▼
       OIDC Provider
            │
            ▼
         IAM Role
            │
            ▼
      AWS Permissions
```

### Security Advantage

Instead of:

```text
❌ AWS Access Key inside container
❌ AWS Secret Key inside container
❌ Hard-coded AWS credentials
```

The project uses:

```text
Kubernetes Service Account
          ↓
      OIDC Trust
          ↓
       IAM Role
          ↓
   Temporary AWS Access
```

---

# 5️⃣ Install AWS Load Balancer Controller

## Add the EKS Helm Repository

```bash
helm repo add eks https://aws.github.io/eks-charts
```

## Update Helm Repository

```bash
helm repo update
```

## Install the Controller

```bash
helm upgrade --install aws-load-balancer-controller \
  eks/aws-load-balancer-controller \
  --namespace kube-system \
  --set clusterName=demo-cluster \
  --set serviceAccount.create=false \
  --set serviceAccount.name=aws-load-balancer-controller \
  --set region=ap-south-2
```

### Verify the Controller

```bash
kubectl get deployment \
  -n kube-system \
  aws-load-balancer-controller
```

### Check Controller Pods

```bash
kubectl get pods -n kube-system
```

### Check Controller Status

```bash
kubectl get pods \
  -n kube-system \
  -l app.kubernetes.io/name=aws-load-balancer-controller
```

> [!NOTE]
> The controller watches Kubernetes resources such as **Ingress** and creates the required AWS load-balancing resources.

---

# 6️⃣ Deploy the 2048 Application

The application deployment contains:

- **Namespace**
- **Deployment**
- **Service**
- **Ingress**

Deploy the application:

```bash
kubectl apply -f \
https://raw.githubusercontent.com/kubernetes-sigs/aws-load-balancer-controller/v2.5.4/docs/examples/2048/2048_full.yaml
```

### Verify Pods

```bash
kubectl get pods -n game-2048
```

### Verify Service

```bash
kubectl get service -n game-2048
```

### Verify Deployment

```bash
kubectl get deployment -n game-2048
```

### View All Resources

```bash
kubectl get all -n game-2048
```

---

# 7️⃣ Verify the Kubernetes Ingress

Check the Ingress resource:

```bash
kubectl get ingress -n game-2048
```

### Example Output

```text
NAME            CLASS   HOSTS   ADDRESS
ingress-2048    alb     *       xxx.ap-south-2.elb.amazonaws.com
```

The **AWS Load Balancer Controller** detects the Ingress and provisions an **AWS Application Load Balancer**.

---

# 🌐 Application Access

After the ALB has been provisioned, retrieve the ALB DNS name:

```bash
kubectl get ingress -n game-2048
```

You should see an address similar to:

```text
xxx.ap-south-2.elb.amazonaws.com
```

Open the address in your browser:

```text
http://<ALB-DNS-NAME>
```

> [!TIP]
> ALB provisioning can take a few minutes. If the `ADDRESS` field is empty immediately after applying the Ingress, wait and run the command again.

---

# 🔍 Verification

## Kubernetes Resources

### Pods

```bash
kubectl get pods -n game-2048
```

### Services

```bash
kubectl get svc -n game-2048
```

### Ingress

```bash
kubectl get ingress -n game-2048
```

### Deployments

```bash
kubectl get deployment -n game-2048
```

---

## AWS Load Balancer Controller

```bash
kubectl get pods \
  -n kube-system \
  -l app.kubernetes.io/name=aws-load-balancer-controller
```

---

## Detailed Ingress Information

```bash
kubectl describe ingress -n game-2048
```

---

## Check All Application Resources

```bash
kubectl get all -n game-2048
```

---

# 📸 Screenshots

## 🏗️ EKS Cluster

Add your EKS cluster screenshot here.

![EKS Cluster](images/eks-cluster.png)

---

## ☸️ Kubernetes Pods

![Kubernetes Pods](images/kubernetes-pods.png)

---

## 🌐 AWS Application Load Balancer

![AWS ALB](images/aws-alb.png)

---

## 🎮 2048 Game

![2048 Game](images/2048-game.png)

---

# 🔄 Complete Deployment Workflow

```text
┌───────────────────────────┐
│   1. Create EKS Cluster   │
└─────────────┬─────────────┘
              │
              ▼
┌───────────────────────────┐
│  2. Configure IAM OIDC    │
└─────────────┬─────────────┘
              │
              ▼
┌───────────────────────────┐
│  3. Create IAM Policy     │
└─────────────┬─────────────┘
              │
              ▼
┌───────────────────────────┐
│ 4. Configure IRSA         │
└─────────────┬─────────────┘
              │
              ▼
┌───────────────────────────┐
│ 5. Install Load Balancer  │
│    Controller using Helm  │
└─────────────┬─────────────┘
              │
              ▼
┌───────────────────────────┐
│ 6. Deploy 2048 App        │
└─────────────┬─────────────┘
              │
              ▼
┌───────────────────────────┐
│ 7. Create Kubernetes      │
│    Ingress                 │
└─────────────┬─────────────┘
              │
              ▼
┌───────────────────────────┐
│ 8. AWS ALB Provisioned    │
└─────────────┬─────────────┘
              │
              ▼
┌───────────────────────────┐
│ 9. Access 2048 Game       │
└───────────────────────────┘
```

---

# 🔐 Security

This project uses **IAM Roles for Service Accounts (IRSA)** to provide AWS permissions to the AWS Load Balancer Controller.

### ❌ Avoid

```text
AWS Access Keys
AWS Secret Keys
Hard-coded credentials
Credentials stored inside containers
```

### ✅ Implemented

```text
Kubernetes Service Account
            │
            ▼
       OIDC Provider
            │
            ▼
         IAM Role
            │
            ▼
      AWS Permissions
```

This provides a cleaner and more secure authentication mechanism between Kubernetes workloads and AWS services.

---

# 🧹 Cleanup

> [!WARNING]
> EKS, Fargate, load balancers, and associated AWS networking resources can generate charges. Delete the resources when you are finished testing.

Delete the EKS cluster:

```bash
eksctl delete cluster \
  --name demo-cluster \
  --region ap-south-2
```

Verify that the cluster has been removed:

```bash
aws eks list-clusters \
  --region ap-south-2
```

Also verify that unwanted AWS resources such as **ALBs, Elastic IPs, NAT Gateways, and other billable resources** have been removed.

---

# 📚 What I Learned

Through this project, I gained practical experience with:

- ☁️ Amazon EKS
- ☸️ Kubernetes
- ⚙️ AWS Fargate
- 🔀 Kubernetes Services
- 🌐 Kubernetes Ingress
- ⚖️ AWS Application Load Balancer
- 🎛️ AWS Load Balancer Controller
- 📦 Helm
- 🔐 IAM
- 🔑 OIDC
- 🛡️ IRSA
- 🏗️ AWS CloudFormation
- 🚀 eksctl
- 🖥️ kubectl
- 🌐 Cloud-native networking
- 🔧 Kubernetes troubleshooting

---

# 🚀 Future Improvements

The project can be extended by implementing:

- [ ] Docker image build pipeline
- [ ] Amazon ECR
- [ ] GitHub Actions CI/CD
- [ ] Terraform infrastructure
- [ ] Automated Kubernetes deployments
- [ ] Prometheus monitoring
- [ ] Grafana dashboards
- [ ] CloudWatch logging
- [ ] HTTPS with AWS Certificate Manager
- [ ] Route 53 custom domain
- [ ] Horizontal Pod Autoscaler
- [ ] Kubernetes Secrets
- [ ] Production monitoring and alerting

---

# 🎯 Skills Demonstrated

```text
AWS
├── EKS
├── Fargate
├── IAM
├── OIDC
├── ALB
└── CloudFormation

Kubernetes
├── Deployment
├── Service
├── Ingress
├── Pods
└── Namespaces

DevOps Tools
├── eksctl
├── kubectl
└── Helm

Security
└── IRSA
```

##output
<img width="1901" height="966" alt="Screenshot 2026-10-04 234423" src="https://github.com/user-attachments/assets/70cefdfe-2b98-41b9-b7b2-f9fe07a67367" />


---


## Rithish Kumar


**Cloud & DevOps Enthusiast**

### Technologies

`AWS` `EKS` `Kubernetes` `Docker` `Terraform` `Helm` `Jenkins` `GitHub Actions` `Python` `Linux`

---


