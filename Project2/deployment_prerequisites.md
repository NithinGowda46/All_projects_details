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

The CSI Driver is a **cluster-level component** and runs in the `kube-system` namespace. It is not installed in the application namespace `chatapp`.

Run from the CD server:

```bash
kubectl apply -f https://raw.githubusercontent.com/kubernetes-sigs/secrets-store-csi-driver/v1.6.0/deploy/secrets-store-csi-driver.yaml
```

The CSI Driver provides the Kubernetes integration required to mount secrets from external secret providers.

In this project, the **AWS Secrets and Configuration Provider (ASCP)** is installed separately as an **EKS add-on** and works together with the Secrets Store CSI Driver to retrieve secrets from AWS Secrets Manager.

The application-specific `SecretProviderClass` is created in the `chatapp` namespace and uses the CSI Driver to retrieve the required secrets.

The architecture is:

```text
EKS Cluster
    │
    ├── kube-system
    │      └── Secrets Store CSI Driver
    │
    └── chatapp
           └── SecretProviderClass
                  ↓
                ASCP
                  ↓
          AWS Secrets Manager
```

For this project, **do not install ASCP again using Helm or the AWS provider YAML**, because ASCP is already installed as an EKS add-on.

The additional RBAC required for synchronizing the mounted secret into a Kubernetes Secret is configured separately in the next section.
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

---

## 4. Configure EKS Pod Identity Association

Create the EKS Pod Identity association from the AWS Console.

Go to:

**AWS Console → EKS → Clusters → `chat-cluster` → Access → Pod Identity associations → Create**

Configure the following:

```text
Namespace:        chatapp
Service account:  backend-sa
IAM role:         ChatAppBackendSecretsRole
```

### ServiceAccount

The backend uses the following Kubernetes ServiceAccount:

```text
backend-sa
```

It is defined in:

```text
k8s/serviceaccount.yaml
```

The backend Deployment uses this ServiceAccount:

```yaml
spec:
  serviceAccountName: backend-sa
```

### IAM Role

The IAM role associated with `backend-sa` is:

```text
ChatAppBackendSecretsRole
```

This IAM role must have permission to read the required secret from AWS Secrets Manager.

The required AWS Secrets Manager secret is:

```text
chatapp/backend
```

The secret contains:

```text
MONGODB_URI
JWT_SECRET
```

The actual secret values must **not** be stored in GitHub.

### Pod Identity Flow

```text
Backend Pod
      ↓
backend-sa
      ↓
EKS Pod Identity Association
      ↓
EKS Pod Identity Agent
      ↓
ChatAppBackendSecretsRole
      ↓
secretsmanager:GetSecretValue
      ↓
AWS Secrets Manager
      ↓
chatapp/backend
```

The backend `SecretProviderClass` uses EKS Pod Identity with:

```yaml
parameters:
  usePodIdentity: "true"
```

This allows the AWS Secrets Manager provider to use the temporary credentials provided through EKS Pod Identity.

Therefore, AWS access keys or secret keys are **not required inside the Kubernetes Deployment**.

-----------------------------------------------------------------

1. EKS Add-ons
   ├── Metrics Server
   ├── ASCP
   └── Pod Identity Agent

2. Secrets Store CSI Driver
   └── kube-system

3. Secret Synchronization RBAC
   └── cluster-level RBAC

4. Pod Identity Association
   └── chatapp/backend-sa → IAM Role

--------------------------------------------------------------------


