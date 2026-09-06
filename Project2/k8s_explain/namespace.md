# `namespace.yaml` — Kubernetes Namespace

## Purpose

Creates the dedicated `chatapp` namespace. Keeping application resources in their own namespace provides logical isolation and makes resource management, RBAC, secrets, services, and troubleshooting easier.

## Complete YAML

```yaml
apiVersion: v1
kind: Namespace
metadata:
  name: chatapp
```

## Explanation

### `apiVersion`

```yaml
apiVersion: v1
```

Uses the Kubernetes core API group.

### `kind`

```yaml
kind: Namespace
```

Tells Kubernetes that this manifest creates a Namespace.

### `metadata.name`

```yaml
name: chatapp
```

Creates the namespace named `chatapp`.

All application resources in this project use this namespace, including Deployments, Services, HPA, ServiceAccount, SecretProviderClass, and Ingress.

## Architecture

```text
EKS Cluster
    │
    └── chatapp namespace
          ├── Backend
          ├── Frontend
          ├── Services
          ├── HPA
          ├── Secrets integration
          └── Ingress
```

## Verify

```bash
kubectl get namespace chatapp
```
