# 🔴 Deployment Prerequisites

This document contains **only the additional configuration required after completing `CI-setup.md` and `CD-setup.md` and before applying the Argo CD application files**.

---

## 1. Add Required EKS Add-ons

From the AWS Console:

**AWS Console → EKS → Clusters → `chat-cluster` → Add-ons → Get more add-ons**

Add the following add-ons:

- **Metrics Server** — provides CPU and memory metrics required by the HPA.
- **AWS Secrets and Configuration Provider (ASCP)** — allows Kubernetes workloads to retrieve secrets from AWS Secrets Manager.
- **Amazon EKS Pod Identity Agent** — provides AWS credentials to pods using EKS Pod Identity associations.

---

## 2. Install Secrets Store CSI Driver

Install the **Secrets Store CSI Driver** on the EKS cluster.

Run from the CD server:

```bash
kubectl apply -f https://raw.githubusercontent.com/kubernetes-sigs/secrets-store-csi-driver/v1.6.0/deploy/secrets-store-csi-driver.yaml
```
---

## 3. Secret Synchronization RBAC

Apply the RBAC required to synchronize the secrets retrieved from AWS Secrets Manager into Kubernetes Secrets.

Run from the CD server:

```bash
kubectl apply -f https://raw.githubusercontent.com/kubernetes-sigs/secrets-store-csi-driver/v1.6.0/deploy/rbac-secretprovidersyncing.yaml
```

This RBAC configuration is **cluster-level** and does not need to be installed in the `chatapp` namespace.

The application namespace is:

```text
chatapp
```

The backend Deployment expects the following Kubernetes Secret:

```text
backend-secret
```

The synchronized Secret contains:

```text
MONGODB_URI
JWT_SECRET
```

The `SecretProviderClass` in the `chatapp` namespace is responsible for defining which AWS Secrets Manager values should be synchronized into `backend-secret`.

The SecretProviderClass used by the backend is:

```text
backend-secret-provider
```

It retrieves the following values from the AWS Secrets Manager secret:

```text
chatapp/backend
```

and synchronizes them into the Kubernetes Secret:

```text
backend-secret
```

The synchronization flow is:

```text
AWS Secrets Manager
        ↓
chatapp/backend
        ↓
SecretProviderClass
        ↓
Secrets Store CSI Driver
        ↓
backend-secret
        ↓
Backend Deployment
```

The backend Deployment reads the values from `backend-secret` using `secretKeyRef`:

```yaml
env:
  - name: MONGODB_URI
    valueFrom:
      secretKeyRef:
        name: backend-secret
        key: MONGODB_URI

  - name: JWT_SECRET
    valueFrom:
      secretKeyRef:
        name: backend-secret
        key: JWT_SECRET
```

The actual secret values must **not** be stored in GitHub.
