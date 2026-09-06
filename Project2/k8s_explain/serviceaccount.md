# `serviceaccount.yaml` — Backend ServiceAccount

## Purpose

Creates the Kubernetes ServiceAccount used by the backend pods. The ServiceAccount is associated with the AWS IAM role through EKS Pod Identity so the backend can access AWS Secrets Manager without storing AWS credentials in the pod.

## Complete YAML

```yaml
apiVersion: v1
kind: ServiceAccount

metadata:
  name: backend-sa
  namespace: chatapp
```

## Explanation

`apiVersion: v1` uses the Kubernetes core API.

`kind: ServiceAccount` creates a ServiceAccount.

`name: backend-sa` gives the ServiceAccount the name `backend-sa`.

`namespace: chatapp` places it in the application namespace.

The backend Deployment references it with:

```yaml
serviceAccountName: backend-sa
```

This identity is used with EKS Pod Identity and the IAM role that permits access to the required AWS Secrets Manager secret.

## Verify

```bash
kubectl get serviceaccount backend-sa -n chatapp
```
