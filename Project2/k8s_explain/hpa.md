# `hpa.yaml` — Horizontal Pod Autoscaler

## Purpose

Automatically scales the backend Deployment between two and five replicas based on CPU and memory utilization.

## Complete YAML

```yaml
apiVersion: autoscaling/v2
kind: HorizontalPodAutoscaler
metadata:
  name: backend-hpa
  namespace: chatapp

spec:
  scaleTargetRef:
    kind: Deployment
    apiVersion: apps/v1
    name: backend-deployment

  minReplicas: 2
  maxReplicas: 5

  metrics:
    - type: Resource
      resource:
        name: cpu
        target:
          type: Utilization
          averageUtilization: 70

    - type: Resource
      resource:
        name: memory
        target:
          type: Utilization
          averageUtilization: 80
```

## Explanation

### API version and kind

```yaml
apiVersion: autoscaling/v2
kind: HorizontalPodAutoscaler
```

Uses the autoscaling v2 API and creates an HPA resource.

### Target Deployment

```yaml
scaleTargetRef:
  kind: Deployment
  apiVersion: apps/v1
  name: backend-deployment
```

The HPA controls the replica count of `backend-deployment`.

### Minimum and maximum replicas

```yaml
minReplicas: 2
maxReplicas: 5
```

The backend always maintains at least two replicas and can scale to a maximum of five replicas.

### CPU metric

```yaml
name: cpu
target:
  type: Utilization
  averageUtilization: 70
```

Targets average CPU utilization of 70% across the backend pods.

### Memory metric

```yaml
name: memory
target:
  type: Utilization
  averageUtilization: 80
```

Targets average memory utilization of 80% across the backend pods.

## Scaling model

```text
                 Backend Deployment
                        │
                   2 replicas
                        │
              ┌─────────┴─────────┐
              │                   │
          CPU > 70%          Memory > 80%
              │                   │
              └─────────┬─────────┘
                        ▼
                  Scale Up
                        │
                   Maximum 5
```

The HPA uses the resource requests defined in the backend Deployment when calculating utilization percentages.

## Verify

```bash
kubectl get hpa backend-hpa -n chatapp
kubectl describe hpa backend-hpa -n chatapp
kubectl get deployment backend-deployment -n chatapp
```
