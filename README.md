# 🚀 Deploying Amazon EKS Cluster on AWS Fargate with ALB Ingress Controller  

This project demonstrates how to deploy an **Amazon Elastic Kubernetes Service (EKS)** cluster using **AWS Fargate**, configure IAM roles for service accounts (IRSA), and expose a sample application (`2048 game`) using the **AWS Load Balancer Controller** and an **Application Load Balancer (ALB)**.  

---

## 📌 Prerequisites  
Before starting, ensure you have the following installed and configured locally:  

- **kubectl** – Kubernetes CLI  
- **eksctl** – CLI for Amazon EKS cluster creation and management  
- **AWS CLI** – For interacting with AWS services (`aws configure` to set up credentials)  

---

## ⚙️ 1. Create EKS Cluster on Fargate  

AWS Fargate is a **serverless compute engine for containers**.  
It removes the need to provision and manage EC2 worker nodes.  

```bash
eksctl create cluster --name demo-cluster --region ap-south-1 --fargate
🔗 2. Configure kubectl
Connect kubectl to your EKS cluster by updating kubeconfig:

bash
Copy code
aws eks --region ap-south-1 update-kubeconfig --name demo-cluster
📝 3. Create a Fargate Profile
Define which workloads run on Fargate:

bash
Copy code
eksctl create fargateprofile \
  --cluster demo-cluster \
  --region ap-south-1 \
  --name alb-sample-app \
  --namespace game-2048
🎮 4. Deploy Sample Application (2048 Game)
Apply the provided Kubernetes manifest:

bash
Copy code
kubectl apply -f https://raw.githubusercontent.com/kubernetes-sigs/aws-load-balancer-controller/v2.5.4/docs/examples/2048/2048_full.yaml
Verify resources:

bash
Copy code
kubectl get pods -n game-2048 -o wide
kubectl get svc -n game-2048
kubectl get ingress -n game-2048
🔐 5. Configure IAM OIDC Provider
EKS needs an OIDC provider to allow IAM Roles for Service Accounts (IRSA):

bash
Copy code
eksctl utils associate-iam-oidc-provider --cluster demo-cluster --approve
🛡️ 6. Create IAM Policy & Role for ALB Controller
Download policy:
bash
Copy code
curl -O https://raw.githubusercontent.com/kubernetes-sigs/aws-load-balancer-controller/v2.11.0/docs/install/iam_policy.json
Create policy:
bash
Copy code
aws iam create-policy \
  --policy-name AWSLoadBalancerControllerIAMPolicy \
  --policy-document file://iam_policy.json
Create IAM service account:
bash
Copy code
eksctl create iamserviceaccount \
  --cluster=demo-cluster \
  --namespace=kube-system \
  --name=aws-load-balancer-controller \
  --role-name AmazonEKSLoadBalancerControllerRole \
  --attach-policy-arn=arn:aws:iam::<your-aws-account-id>:policy/AWSLoadBalancerControllerIAMPolicy \
  --approve
🌐 7. Deploy AWS Load Balancer Controller
Add Helm repo:
bash
Copy code
helm repo add eks https://aws.github.io/eks-charts
Install controller:
bash
Copy code
helm install aws-load-balancer-controller eks/aws-load-balancer-controller -n kube-system \
  --set clusterName=demo-cluster \
  --set serviceAccount.create=false \
  --set serviceAccount.name=aws-load-balancer-controller \
  --set region=ap-south-1 \
  --set vpcId=<your-vpc-id>
