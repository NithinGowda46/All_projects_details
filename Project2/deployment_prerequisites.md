# 🔴 Deployment Prerequisites

This document contains **only the additional configuration required after completing `CI-setup.md` and `CD-setup.md` and before applying the Argo CD application files**.

---

## 1. EKS Public Subnet Tags

Because the application uses an **internet-facing AWS Application Load Balancer**, tag the public subnets used by the EKS cluster:

```text
Key:   kubernetes.io/role/elb
Value: 1
```

---

## 2. Add Required EKS Add-ons

From the AWS Console:

**AWS Console → EKS → Clusters → `chat-cluster` → Add-ons → Get more add-ons**

Add the required add-ons:

- **Metrics Server** — provides CPU and memory metrics required by the HPA.
- **AWS Secrets Manager and Configuration Provider (ASCP)** — allows Kubernetes workloads to retrieve secrets from AWS Secrets Manager through the Secrets Store CSI integration.
- **Amazon EKS Pod Identity Agent** — provides AWS credentials to pods using EKS Pod Identity associations.

---

## 3. AWS Secrets Store CSI Driver

Install the **Secrets Store CSI Driver** and the **AWS Secrets Manager provider** on the EKS cluster.

These components allow the Kubernetes `SecretProviderClass` to retrieve secrets from AWS Secrets Manager.

---

## 4. Secret Synchronization RBAC

Apply the RBAC required for synchronizing external secrets into Kubernetes Secrets:

```bash
kubectl apply -f https://raw.githubusercontent.com/kubernetes-sigs/secrets-store-csi-driver/v1.6.0/deploy/rbac-secretprovidersyncing.yaml
```

The backend Deployment expects the synchronized Kubernetes Secret:

```text
backend-secret
```

---

## 5. Backend Pod Identity Association

Create the EKS Pod Identity association for the backend.

In the AWS Console:

**EKS → Clusters → `chat-cluster` → Access → Pod Identity associations → Create**

Use:

```text
Namespace:      chatapp
Service account: backend-sa
IAM role:       ChatAppBackendSecretsRole
```

The ServiceAccount is defined in the CD repository as:

```text
k8s/serviceaccount.yaml
```

The IAM role must have permission to read the backend secret from AWS Secrets Manager.

The complete flow is:

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
```

The backend `SecretProviderClass` uses Pod Identity with:

```yaml
usePodIdentity: "true"
```

The secret used by the application is:

```text
chatapp/backend
```

It contains:

```text
MONGODB_URI
JWT_SECRET
```

The actual secret values must not be stored in GitHub.

---

## 6. DNS Configuration

After the Application Load Balancer is created by the Kubernetes Ingress, configure DNS so the application can be accessed using the required domain name.

Create a DNS record in **Amazon Route 53** pointing the application domain to the ALB.

For an ALB, use an **Alias record** rather than manually entering the ALB IP address because the ALB IP addresses can change.

Example:

```text
Record name:  chat.example.com
Record type:  A
Alias:        Yes
Target:       Application Load Balancer
```

If an internal/private DNS name is required for resources in the VPC, create the corresponding private Route 53 record and point it to the required private resource/IP.

For example, a private record can resolve to an internal address such as:

```text
10.18.x.x
```

Use the actual private IP address assigned to the required resource; do not use a hard-coded IP if the resource's private IP can change.

---

## 7. Tag the Public IP / Elastic IP

If an **Elastic IP** is used for a public-facing AWS resource such as the NAT Gateway, add a meaningful Name tag so the resource can be easily identified.

In the AWS Console:

**EC2 → Network & Security → Elastic IPs → Select Elastic IP → Tags → Manage tags**

Example:

```text
Key:   Name
Value: chatapp-nat-eip
```

Use an appropriate tag for the actual resource if the public IP belongs to another component.

---

## 8. Deployment Order

After `CI-setup.md` and `CD-setup.md` are complete, follow this order:

```text
1. Configure EKS public subnet tags
        ↓
2. Add Metrics Server, AWS Secrets Manager and Configuration Provider,
   and EKS Pod Identity Agent add-ons
        ↓
3. Install Secrets Store CSI Driver + AWS provider
        ↓
4. Apply secret synchronization RBAC
        ↓
5. Configure EKS Pod Identity association
        ↓
6. Configure DNS in Route 53
        ↓
7. Tag the required public/Elastic IP
        ↓
8. Argo CD deployment
```

Once these additional prerequisites are complete, proceed with the **Argo CD deployment**.