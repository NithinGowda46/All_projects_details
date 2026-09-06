# `ingress.yaml` — AWS Application Load Balancer Ingress

## Purpose

Defines how external HTTP traffic reaches the Chat Application through an AWS Application Load Balancer.

The current configuration creates an **internet-facing ALB** and routes `/api` to the backend Service and `/` to the frontend Service.

## Complete YAML

```yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: chatapp-ingress
  namespace: chatapp
  annotations:
    alb.ingress.kubernetes.io/scheme: internet-facing

spec:
  ingressClassName: alb

  rules:
    - http:
        paths:

          # Backend API
          - path: /api
            pathType: Prefix
            backend:
              service:
                name: backend-service
                port:
                  number: 5001

          # Frontend
          - path: /
            pathType: Prefix
            backend:
              service:
                name: frontend-service
                port:
                  number: 80
```

## Explanation

### API version and kind

```yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
```

Uses the Kubernetes networking API and creates an Ingress resource.

### Name and namespace

```yaml
name: chatapp-ingress
namespace: chatapp
```

Creates `chatapp-ingress` in the `chatapp` namespace.

### Internet-facing ALB

```yaml
annotations:
  alb.ingress.kubernetes.io/scheme: internet-facing
```

Requests an internet-facing Application Load Balancer. The ALB is therefore reachable from the public internet rather than only through the private VPC/VPN path.

The public ALB uses public subnets that are appropriately tagged for EKS Auto Mode.

### IngressClass

```yaml
ingressClassName: alb
```

References the `alb` IngressClass defined in `ingressclass.yaml`.

### Routing rules

The Ingress defines path-based routing.

#### Backend API

```yaml
- path: /api
  pathType: Prefix
  backend:
    service:
      name: backend-service
      port:
        number: 5001
```

Requests beginning with `/api` are forwarded to:

```text
backend-service:5001
```

#### Frontend

```yaml
- path: /
  pathType: Prefix
  backend:
    service:
      name: frontend-service
      port:
        number: 80
```

Requests beginning with `/` are forwarded to:

```text
frontend-service:80
```

## Traffic flow

```text
                         Internet
                            │
                            ▼
                 Internet-Facing ALB
                            │
                  ┌─────────┴─────────┐
                  │                   │
                /api                  /
                  │                   │
                  ▼                   ▼
         backend-service       frontend-service
              :5001                   :80
                  │                   │
                  ▼                   ▼
           Backend Pods         Frontend Pods
```

## Current HTTP behavior

The current manifest configures ALB scheme only. HTTPS/ACM listener configuration is not present in this YAML yet.

## Verify

```bash
kubectl get ingress chatapp-ingress -n chatapp
kubectl describe ingress chatapp-ingress -n chatapp
```

The Ingress status should show the AWS Load Balancer hostname after provisioning completes.
