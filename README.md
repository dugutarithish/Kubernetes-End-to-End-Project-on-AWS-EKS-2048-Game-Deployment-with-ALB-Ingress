# Kubernetes-End-to-End-Project-on-AWS-EKS-2048-Game-Deployment-with-ALB-Ingress


An end-to-end DevOps project demonstrating how to deploy a containerized application on Amazon Elastic Kubernetes Service (EKS), manage traffic using Kubernetes Ingress, and automate external routing via the AWS Load Balancer Controller.

🚀 Project Overview
This project showcases production-grade Kubernetes networking and cloud-native architecture on AWS. It provisions an EKS cluster, configures IAM roles for service accounts (IRSA), deploys the AWS Load Balancer Controller, and exposes a containerized application to the public internet via an AWS Application Load Balancer (ALB).

🛠️ Tech Stack & Tools Used
Cloud Provider: Amazon Web Services (AWS) (ap-south-2)

Orchestration: Amazon EKS, eksctl, kubectl

Networking & Ingress: AWS Load Balancer Controller, Kubernetes Ingress, Target Group Bindings

Infrastructure as Code / Scripting: AWS CloudFormation, IAM OIDC

Application: 2048 Game Deployment (k8s-game2048)

Architecture & Deployment Steps
1. Create the EKS Cluster
Deploy a managed EKS cluster with dedicated VPC and networking configurations using
<img width="1024" height="520" alt="image" src="https://github.com/user-attachments/assets/3baf5548-8af5-48cf-84e2-b151e78e304f" />
2. Set Up AWS Load Balancer Controller
Created an IAM policy (AWSLoadBalancerControllerIAMPolicy) to allow Kubernetes to manage AWS ALBs.

Associated an IAM OIDC provider with the cluster.

Deployed the AWS Load Balancer Controller via Helm / YAML manifests.

3. Deploy the Application & Ingress Resources
Applied the Kubernetes manifests which configure:

Deployment & Service: Running the game pods inside the cluster.

Ingress Resource: Triggering the creation of an external AWS Application Load Balancer (ALB).

# AWS EKS Game 2048 Ingress Project

A production-grade, end-to-end Kubernetes project deploying the classic **2048 game** on **Amazon EKS**, leveraging **AWS Fargate**, **IAM Roles for Service Accounts (IRSA)**, and the **AWS Load Balancer Controller** to manage public HTTP traffic via an AWS Application Load Balancer (ALB).



<img width="1901" height="966" alt="Screenshot 2026-10-04 234423" src="https://github.com/user-attachments/assets/fb8435ab-1502-44d7-aa00-5727bc3c4f90" />

