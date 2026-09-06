# 🔴 Deployment Prerequisites

This document contains **only the additional configuration required after completing `CI-setup.md` and `CD-setup.md` and before applying the Argo CD application files**.

---

## 1. EKS Public Subnet Tags

Because the application uses an **internet-facing AWS Application Load Balancer**, tag the public subnets used by the EKS cluster:

```text
Key:   kubernetes.io/role/elb
Value: 1
```

Verify the subnet tags in:

**AWS Console → VPC → Subnets → Tags**

---

## 2. AWS Secrets Store CSI Driver

Install the **Secrets Store CSI Driver** and the **AWS Secrets Manager provider/add-on** on the EKS cluster.

Verify:

```bash
kubectl get pods -n aws-secrets-manager
```

Verify the CRD:

```bash
kubectl get crd secretproviderclasses.secrets-store.csi.x-k8s.io
```

The driver and AWS provider must be running before the backend Deployment is synchronized.

---

## 3. Secret Synchronization RBAC

Apply the RBAC required for synchronizing external secrets into Kubernetes Secrets:

```bash
kubectl apply -f https://raw.githubusercontent.com/kubernetes-sigs/secrets-store-csi-driver/v1.6.0/deploy/rbac-secretprovidersyncing.yaml
```

Verify:

```bash
kubectl get clusterrole secretprovidersyncing-role
kubectl get clusterrolebinding secretprovidersyncing-rolebinding
```

The backend Deployment expects the synchronized Kubernetes Secret:

```text
backend-secret
```

---

## 4. Backend Pod Identity Association

Create the EKS Pod Identity association for the backend:

```text
Namespace:      chatapp
ServiceAccount: backend-sa
IAM Role:       ChatAppBackendSecretsRole
```

The required ServiceAccount is defined in the CD repository as:

```text
k8s/serviceaccount.yaml
```

Verify the association in:

**AWS Console → EKS → Clusters → chat-cluster → Pod Identity associations**

The resulting flow is:

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

---

## 5. Verify AWS Secret Access

Confirm that the secret used by the backend exists in AWS Secrets Manager:

```text
chatapp/backend
```

It must contain:

```text
MONGODB_URI
JWT_SECRET
```

Do not put the actual secret values in GitHub.

The backend `SecretProviderClass` will retrieve these values when the pod starts.

---

## 6. Verify HPA Metrics

The backend HPA requires resource metrics.

Verify:

```bash
kubectl top pods -n chatapp
```

If metrics are unavailable, install/configure a compatible metrics provider before relying on the HPA.

The current HPA configuration scales the backend based on CPU and memory utilization.

---

## 7. Verify EKS ALB Readiness

EKS Auto Mode provides the AWS load-balancing capability required by the application's ALB Ingress.

Verify that the cluster is ready to provision an ALB and that the public subnet tags are correct.

The application Ingress is configured as:

```text
Scheme: internet-facing
```

Traffic will be routed as:

```text
/api → backend-service:5001
/    → frontend-service:80
```

No public IP is required on the application pods or Services because the ALB is the external entry point.

---

## 8. Verify CD Repository

Confirm that the latest Kubernetes and Argo CD configuration is pushed to:

```text
Project2-chatapp-CD
```

Required directories:

```text
k8s/
argocd/
```

The `k8s/` directory must contain the current application manifests, and the `argocd/` directory must contain:

```text
project.yaml
application.yaml
```

---

## 9. Argo CD Repository Access

The Argo CD Application points to the GitHub CD repository and the `k8s/` directory.

For the current public repository, no GitHub repository credential is required.

Before creating the Argo CD Application, verify from the CD server that the repository is reachable:

```bash
git ls-remote https://github.com/NithinGowda46/Project2-chatapp-CD.git
```

---

## 10. Final Verification Before Argo CD

Run the following checks from the CD server:

### EKS access

```bash
kubectl get nodes
```

### Secrets Store CSI Driver

```bash
kubectl get pods -n aws-secrets-manager
```

### SecretProviderClass CRD

```bash
kubectl get crd secretproviderclasses.secrets-store.csi.x-k8s.io
```

### HPA metrics

```bash
kubectl top pods -n chatapp
```

### GitHub repository access

```bash
git ls-remote https://github.com/NithinGowda46/Project2-chatapp-CD.git
```

---

## 11. Deployment Order

After `CI-setup.md` and `CD-setup.md` are complete, follow this order:

```text
1. Configure EKS public subnet tags
        ↓
2. Install Secrets Store CSI Driver + AWS provider
        ↓
3. Apply secret synchronization RBAC
        ↓
4. Configure EKS Pod Identity association
        ↓
5. Verify AWS Secrets Manager access
        ↓
6. Verify HPA metrics
        ↓
7. Verify ALB readiness
        ↓
8. Verify GitHub CD repository
        ↓
9. Apply Argo CD project.yaml
        ↓
10. Apply Argo CD application.yaml
        ↓
11. Argo CD synchronizes the k8s/ directory
        ↓
12. Verify Chat Application
```

Once these additional prerequisites are complete, proceed with the **Argo CD deployment**.