# `frontend_deployment.yaml` — Frontend Deployment

## Purpose

Creates and manages the frontend application pods. The frontend image is stored in Amazon ECR and is served on port `80`.

## Complete YAML

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: frontend-deployment
  namespace: chatapp

spec:
  replicas: 2

  selector:
    matchLabels:
      app: frontend

  template:
    metadata:
      labels:
        app: frontend

    spec:
      containers:
        - name: frontend-container
          image: 236209348145.dkr.ecr.ap-southeast-1.amazonaws.com/chat-app-frontend:7

          ports:
            - containerPort: 80

          resources:
            requests:
              cpu: "100m"
              memory: "128Mi"

            limits:
              cpu: "300m"
              memory: "256Mi"

          startupProbe:
            httpGet:
              path: /
              port: 80
            periodSeconds: 10
            failureThreshold: 30

          livenessProbe:
            httpGet:
              path: /
              port: 80
            initialDelaySeconds: 10
            periodSeconds: 10
            failureThreshold: 3

          readinessProbe:
            httpGet:
              path: /
              port: 80
            initialDelaySeconds: 5
            periodSeconds: 5
            failureThreshold: 3
```

## Explanation

The Deployment is named `frontend-deployment` and runs in the `chatapp` namespace.

### Replicas

```yaml
replicas: 2
```

Maintains two frontend pods for availability.

### Selector and labels

```yaml
selector:
  matchLabels:
    app: frontend
```

and:

```yaml
labels:
  app: frontend
```

Connect the Deployment to pods carrying the `app=frontend` label. The frontend Service uses the same label.

### ECR image

```yaml
image: 236209348145.dkr.ecr.ap-southeast-1.amazonaws.com/chat-app-frontend:7
```

Pulls frontend image tag `7` from Amazon ECR.

### Container port

```yaml
containerPort: 80
```

The frontend web server listens on port `80`.

### Resources

```yaml
requests:
  cpu: "100m"
  memory: "128Mi"
limits:
  cpu: "300m"
  memory: "256Mi"
```

Requests reserve the minimum scheduling resources and limits cap container usage.

### Startup probe

Checks `/` on port `80` every 10 seconds while the frontend starts.

### Liveness probe

Checks `/` every 10 seconds after a 10-second initial delay. Repeated failures indicate that the container is unhealthy.

### Readiness probe

Checks `/` every 5 seconds after a 5-second initial delay. Only ready frontend pods receive Service traffic.

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
kubectl get deployment frontend-deployment -n chatapp
kubectl get pods -n chatapp -l app=frontend
kubectl describe deployment frontend-deployment -n chatapp
```
