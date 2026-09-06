# `backend_service.yaml` — Backend Service

## Purpose

Provides stable internal networking for the backend pods. The Service is `ClusterIP`, so the backend is not directly exposed to the internet.

## Complete YAML

```yaml
apiVersion: v1
kind: Service
metadata:
  name: backend-service
  namespace: chatapp

spec:
  type: ClusterIP

  selector:
    app: backend

  ports:
    - protocol: TCP
      port: 5001
      targetPort: 5001
```

## Explanation

`apiVersion: v1` uses the Kubernetes core API.

`kind: Service` creates a Service.

`name: backend-service` gives it a stable DNS name and Service identity.

`namespace: chatapp` places it in the application namespace.

### ClusterIP

```yaml
type: ClusterIP
```

Makes the Service reachable inside the Kubernetes cluster only. External traffic reaches the backend through the ALB Ingress rather than directly through this Service.

### Selector

```yaml
selector:
  app: backend
```

Selects the backend pods created by `backend_deployment.yaml`.

### Ports

```yaml
port: 5001
targetPort: 5001
```

The Service accepts traffic on port `5001` and forwards it to port `5001` on the selected backend pods.

## Traffic flow

```text
Internet
   ↓
Internet-Facing ALB
   ↓ /api
backend-service:5001
   ↓
Backend Pods:5001
```

## Verify

```bash
kubectl get service backend-service -n chatapp
kubectl describe service backend-service -n chatapp
kubectl get endpoints backend-service -n chatapp
```
