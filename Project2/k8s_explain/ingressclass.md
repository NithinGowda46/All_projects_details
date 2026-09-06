# `ingressclass.yaml` — AWS ALB IngressClass

## Purpose

Defines the Kubernetes IngressClass named `alb` and tells EKS Auto Mode to handle Ingress resources using the AWS ALB controller.

## Complete YAML

```yaml
apiVersion: networking.k8s.io/v1
kind: IngressClass
metadata:
  name: alb
spec:
  controller: eks.amazonaws.com/alb
```

## Explanation

### API version

```yaml
apiVersion: networking.k8s.io/v1
```

Uses the Kubernetes networking API.

### Kind

```yaml
kind: IngressClass
```

Creates an IngressClass resource.

### Name

```yaml
name: alb
```

The class is named `alb`.

### Controller

```yaml
controller: eks.amazonaws.com/alb
```

Associates the class with the AWS ALB controller used by EKS Auto Mode.

The application Ingress references this class with:

```yaml
ingressClassName: alb
```

This tells Kubernetes which controller should process the application Ingress.

## Relationship

```text
Ingress
   │
   │ ingressClassName: alb
   ▼
IngressClass: alb
   │
   ▼
AWS ALB Controller / EKS Auto Mode
   │
   ▼
Application Load Balancer
```

## Verify

```bash
kubectl get ingressclass
kubectl describe ingressclass alb
```
