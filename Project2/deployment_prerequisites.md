# 🔴 Deployment Prerequisites

This document contains all prerequisites that must be completed **before applying the Argo CD deployment files** for the Chat Application on Amazon EKS.

---

## 1. Amazon EKS Cluster

Create the EKS cluster with **EKS Auto Mode enabled**.

Verify:

```bash
aws eks describe-cluster \
  --name chat-cluster \
  --region ap-southeast-1 \
  --query "cluster.status"
```

Expected:

```text
"ACTIVE"
```

---

## 2. Public Subnet Configuration

The application uses an **internet-facing AWS Application Load Balancer**.

The public subnets used by EKS must have:

```text
Key:   kubernetes.io/role/elb
Value: 1
```

Public subnets:

```text
1a_public_subnet
1b_public_subnet
1c_public_subnet
```

---

## 3. Amazon ECR

Create the required ECR repositories:

```text
chat-app-backend
chat-app-frontend
```

The required application images must be available in ECR before the Kubernetes Deployments are synchronized by Argo CD.

Verify:

```bash
aws ecr describe-repositories --region ap-southeast-1

aws ecr list-images \
  --repository-name chat-app-backend \
  --region ap-southeast-1

aws ecr list-images \
  --repository-name chat-app-frontend \
  --region ap-southeast-1
```

---

## 4. CD Server

A dedicated CD server is used for Kubernetes and Argo CD operations.

```text
Name:          CD-mainserver
Instance Type: m7i-flex.large
vCPU:          2
Memory:        8 GB
Subnet:        1b_private_subnet
Private IP:    10.18.35.188
Public IPv4:   None
OS:            Amazon Linux 2023
IMDSv2:        Required
```

---

## 5. Required Tools

Install the following tools on the CD server:

- AWS CLI
- kubectl
- Argo CD CLI
- MongoDB Shell (`mongosh`)

Verify:

```bash
aws --version
kubectl version --client
argocd version --client
mongosh --version
```

---

## 6. CD Server IAM Role

Attach:

```text
CD-mainserver-role
```

to the CD server EC2 instance.

Required EKS permission:

```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Allow",
      "Action": [
        "eks:DescribeCluster"
      ],
      "Resource": "*"
    }
  ]
}
```

---

## 7. EKS Access Entry

Add `CD-mainserver-role` as an EKS Access Entry.

```text
Type: Standard
```

Associate:

```text
AmazonEKSClusterAdminPolicy
```

with:

```text
Access Scope: Cluster
```

This allows the CD server to authenticate and perform Kubernetes operations on the EKS cluster.

---

## 8. Configure EKS Access

On the CD server:

```bash
aws eks update-kubeconfig \
  --region ap-southeast-1 \
  --name chat-cluster
```

Verify:

```bash
kubectl get nodes
```

---

## 9. AWS Secrets Manager

Create the application secret:

```text
Secret Name: chatapp/backend
```

Store:

```json
{
  "MONGODB_URI": "<MONGODB_CONNECTION_STRING>",
  "JWT_SECRET": "<STRONG_JWT_SECRET>"
}
```

Generate a strong JWT secret:

```bash
openssl rand -base64 32
```

Never commit the actual secret values to GitHub.

---

## 10. Backend IAM Role

Create/use:

```text
ChatAppBackendSecretsRole
```

The role is used by backend pods through **EKS Pod Identity**.

It must have permission to read the application secret from AWS Secrets Manager.

Example least-privilege policy:

```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Allow",
      "Action": [
        "secretsmanager:GetSecretValue"
      ],
      "Resource": "<CHATAPP_BACKEND_SECRET_ARN>"
    }
  ]
}
```

---

## 11. Backend ServiceAccount

The backend uses:

```text
Namespace:      chatapp
ServiceAccount: backend-sa
```

It is defined in `serviceaccount.yaml` and referenced by the backend Deployment.

```yaml
serviceAccountName: backend-sa
```

---

## 12. EKS Pod Identity

Create an EKS Pod Identity association between:

```text
Namespace:      chatapp
ServiceAccount: backend-sa
IAM Role:       ChatAppBackendSecretsRole
```

The access flow is:

```text
Backend Pod
    ↓
backend-sa
    ↓
EKS Pod Identity
    ↓
ChatAppBackendSecretsRole
    ↓
AWS Secrets Manager
    ↓
chatapp/backend
```

This avoids storing AWS access keys inside the Kubernetes application.

---

## 13. Secrets Store CSI Driver and AWS Provider

The backend retrieves secrets from AWS Secrets Manager using the **Secrets Store CSI Driver** and the AWS provider.

Install the required Secrets Store CSI Driver and AWS Secrets Manager provider/add-on before deploying the backend.

Verify:

```bash
kubectl get pods -n aws-secrets-manager
```

Verify the CRD:

```bash
kubectl get crd secretproviderclasses.secrets-store.csi.x-k8s.io
```

The driver and AWS provider must be running successfully.

---

## 14. Secret Synchronization RBAC

The Secrets Store CSI Driver requires RBAC permissions to synchronize the external secret into a Kubernetes Secret.

Apply:

```bash
kubectl apply -f https://raw.githubusercontent.com/kubernetes-sigs/secrets-store-csi-driver/v1.6.0/deploy/rbac-secretprovidersyncing.yaml
```

Verify:

```bash
kubectl get clusterrole secretprovidersyncing-role
kubectl get clusterrolebinding secretprovidersyncing-rolebinding
```

This allows the secret values to be synchronized into:

```text
backend-secret
```

---

## 15. AWS Secrets to Kubernetes Secret Flow

The Kubernetes file `sectes.yaml` defines the `SecretProviderClass`:

```text
backend-secret-provider
```

It reads:

```text
MONGODB_URI
JWT_SECRET
```

from:

```text
AWS Secrets Manager
        ↓
chatapp/backend
        ↓
Secrets Store CSI Driver
        ↓
SecretProviderClass
        ↓
backend-secret
        ↓
Backend Pod
```

The actual secret values are never stored in the Git repository.

---

## 16. MongoDB Connectivity

The backend requires connectivity to MongoDB.

Verify connectivity from the CD server:

```bash
mongosh "<MONGODB_CONNECTION_STRING>"
```

A successful connection confirms that the database endpoint, credentials, and required network connectivity are working.

The connection string must be stored in AWS Secrets Manager under:

```text
chatapp/backend
```

as:

```text
MONGODB_URI
```

---

## 17. Backend Health Endpoint

The backend Deployment uses:

```text
/health
```

for:

- Startup Probe
- Liveness Probe
- Readiness Probe

The backend must provide:

```text
GET /health
```

on port:

```text
5001
```

---

## 18. Kubernetes Resource Requirements

Backend:

```text
CPU Request:    100m
Memory Request: 256Mi
CPU Limit:      500m
Memory Limit:   512Mi
```

Frontend:

```text
CPU Request:    100m
Memory Request: 128Mi
CPU Limit:      300m
Memory Limit:   256Mi
```

These requests are important because the HPA uses resource utilization for autoscaling.

---

## 19. HPA Metrics

The backend HPA is configured for:

```text
Minimum replicas: 2
Maximum replicas: 5
CPU target:       70%
Memory target:    80%
```

Verify resource metrics:

```bash
kubectl top pods -n chatapp
```

Metrics must be available for resource-based HPA decisions.

---

## 20. AWS ALB Prerequisites

The application uses the AWS Application Load Balancer through:

```text
ingressclass.yaml
ingress.yaml
```

The Ingress uses:

```yaml
alb.ingress.kubernetes.io/scheme: internet-facing
```

Traffic routing:

```text
/api → backend-service:5001
/    → frontend-service:80
```

Both Services remain `ClusterIP`. The ALB provides the external entry point.

---

## 21. GitHub CD Repository

The Kubernetes manifests are maintained in:

```text
Project2-chatapp-CD
```

Required directory:

```text
k8s/
```

Argo CD reads the Kubernetes manifests from this directory.

---

## 22. Argo CD

Argo CD must be installed and available before applying the Argo CD configuration files.

Verify:

```bash
argocd version --client
```

The Argo CD configuration is maintained in:

```text
argocd/
```

The Argo CD Application points to the Kubernetes manifests in:

```text
k8s/
```

---

## 23. Final Pre-Deployment Verification

Before applying the Argo CD files, verify:

### EKS

```bash
kubectl get nodes
```

### Namespace

```bash
kubectl get namespace chatapp
```

### ServiceAccount

```bash
kubectl get serviceaccount backend-sa -n chatapp
```

### Secrets Store CSI Driver

```bash
kubectl get pods -n aws-secrets-manager
```

### SecretProviderClass CRD

```bash
kubectl get crd secretproviderclasses.secrets-store.csi.x-k8s.io
```

### Resource Metrics

```bash
kubectl top pods -n chatapp
```

### Argo CD CLI

```bash
argocd version --client
```

### MongoDB

```bash
mongosh "<MONGODB_CONNECTION_STRING>"
```

---

## 24. Deployment Readiness Checklist

Before applying the Argo CD files:

```text
☐ EKS cluster is ACTIVE
☐ EKS Auto Mode is enabled
☐ Public subnets have kubernetes.io/role/elb=1
☐ Backend ECR image is available
☐ Frontend ECR image is available
☐ CD-mainserver is configured
☐ CD-mainserver-role is attached
☐ EKS Access Entry is configured
☐ kubectl can access the EKS cluster
☐ chatapp/backend exists in AWS Secrets Manager
☐ ChatAppBackendSecretsRole exists
☐ EKS Pod Identity association is configured
☐ backend-sa is configured
☐ Secrets Store CSI Driver is installed
☐ AWS Secrets Manager provider is installed
☐ Secret synchronization RBAC is configured
☐ MongoDB connectivity is working
☐ Backend /health endpoint is available
☐ Resource metrics are available
☐ Argo CD is installed
☐ GitHub CD repository contains the required manifests
```

Once all prerequisites are complete, proceed to the **Argo CD Deployment** section and apply the Argo CD configuration files.
