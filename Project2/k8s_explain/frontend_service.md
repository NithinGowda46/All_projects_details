# `frontend_service.yaml` — Frontend Service

## Purpose

Provides stable internal networking for the frontend pods. The Service uses `ClusterIP` and is reached by the AWS Application Load Balancer through the Ingress.

## Complete YAML

```yaml
apiVersion: v1
kind: Service
metadata:
  name: frontend-service
  namespace: chatapp

spec:
  type: ClusterIP

  selector:
    app: frontend

  ports:
    - protocol: TCP
      port: 80
      targetPort: 80
```

## Explanation

`apiVersion: v1` uses the Kubernetes core API.

`kind: Service` creates a Kubernetes Service.

`name: frontend-service` gives the frontend a stable Service name.

`namespace: chatapp` places it in the application namespace.

### ClusterIP

```yaml
type: ClusterIP
```

Keeps the frontend Service internal to the cluster. The application is publicly exposed through the ALB, not by making the Service a LoadBalancer.

### Selector

```yaml
selector:
  app: frontend
```

Selects the frontend pods created by `frontend_deployment.yaml`.

### Ports

```yaml
port: 80
targetPort: 80
```

Traffic arriving at the Service on port `80` is forwarded to port `80` on the frontend pods.

## Traffic flow

```text
Internet
   ↓
Internet-Facing ALB
   ↓ /
frontend-service:80
   ↓
Frontend Pods:80
```

## Verify

```bash
kubectl get service frontend-service -n chatapp
kubectl describe service frontend-service -n chatapp
kubectl get endpoints frontend-service -n chatapp
```
