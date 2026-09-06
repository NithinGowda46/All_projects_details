# `sectes.yaml` — SecretProviderClass

> The repository filename is `sectes.yaml`.

## Purpose

Defines how the backend retrieves application secrets from AWS Secrets Manager using the AWS provider for the Secrets Store CSI Driver and EKS Pod Identity.

The file also defines synchronization of the external values into a Kubernetes Secret named `backend-secret`.

## Complete YAML

```yaml
apiVersion: secrets-store.csi.x-k8s.io/v1
kind: SecretProviderClass

metadata:
  name: backend-secret-provider
  namespace: chatapp

spec:
  provider: aws

  parameters:
    usePodIdentity: "true"

    objects: |
      - objectName: "chatapp/backend"
        objectType: secretsmanager
        jmesPath:
          - path: MONGODB_URI
            objectAlias: MONGODB_URI
          - path: JWT_SECRET
            objectAlias: JWT_SECRET

  secretObjects:
    - secretName: backend-secret
      type: Opaque
      data:
        - objectName: MONGODB_URI
          key: MONGODB_URI
        - objectName: JWT_SECRET
          key: JWT_SECRET
```

## Explanation

### API and resource type

```yaml
apiVersion: secrets-store.csi.x-k8s.io/v1
kind: SecretProviderClass
```

Uses the Secrets Store CSI Driver API to define an external secret source.

### Name and namespace

```yaml
name: backend-secret-provider
namespace: chatapp
```

Creates `backend-secret-provider` in the `chatapp` namespace.

### AWS provider

```yaml
provider: aws
```

Uses the AWS Secrets Manager provider for the Secrets Store CSI Driver.

### EKS Pod Identity

```yaml
usePodIdentity: "true"
```

The pod uses EKS Pod Identity instead of storing AWS access keys in Kubernetes. The `backend-sa` ServiceAccount is associated with an IAM role that has permission to read the required AWS Secrets Manager secret.

### Secrets Manager object

```yaml
objects: |
  - objectName: "chatapp/backend"
    objectType: secretsmanager
```

Reads the AWS Secrets Manager secret named:

```text
chatapp/backend
```

### JMESPath values

```yaml
jmesPath:
  - path: MONGODB_URI
    objectAlias: MONGODB_URI
  - path: JWT_SECRET
    objectAlias: JWT_SECRET
```

Extracts the `MONGODB_URI` and `JWT_SECRET` fields from the secret and exposes them using the same aliases.

### Kubernetes Secret synchronization

```yaml
secretObjects:
  - secretName: backend-secret
    type: Opaque
```

Synchronizes the selected values into a Kubernetes Secret named `backend-secret`.

The values are mapped as:

```text
MONGODB_URI → backend-secret.MONGODB_URI
JWT_SECRET  → backend-secret.JWT_SECRET
```

The backend Deployment consumes them using `secretKeyRef`.

## Complete secret flow

```text
AWS Secrets Manager
        │
        │ chatapp/backend
        ▼
   EKS Pod Identity
        │
        ▼
Secrets Store CSI Driver
        │
        ▼
SecretProviderClass
        │
        ▼
 Kubernetes Secret
  backend-secret
        │
        ▼
 Backend Deployment
```

## Important

The actual secret values should not be stored in GitHub. This manifest contains only the secret name and keys, not the database password or JWT secret value.

## Verify

```bash
kubectl get secretproviderclass -n chatapp
kubectl describe secretproviderclass backend-secret-provider -n chatapp
kubectl get secret backend-secret -n chatapp
```
