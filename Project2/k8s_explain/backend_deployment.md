# `backend_deployment.yaml` — Backend Deployment

## Purpose

Creates and manages the backend application pods. The Deployment runs two replicas, pulls the backend image from Amazon ECR, injects secrets from the Kubernetes Secret, configures resource requests and limits, performs health checks, and mounts the Secrets Store CSI volume.

## Complete YAML

```yaml
apiVersion: apps/v1
kind: Deployment

metadata:
  name: backend-deployment
  namespace: chatapp

spec:
  replicas: 2

  selector:
    matchLabels:
      app: backend

  template:
    metadata:
      labels:
        app: backend

    spec:
      serviceAccountName: backend-sa

      containers:
        - name: backend-container
          image: 236209348145.dkr.ecr.ap-southeast-1.amazonaws.com/chat-app-backend:7

          ports:
            - containerPort: 5001

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

            - name: PORT
              value: "5001"

            - name: NODE_ENV
              value: "production"

          resources:
            requests:
              cpu: "100m"
              memory: "256Mi"

            limits:
              cpu: "500m"
              memory: "512Mi"

          startupProbe:
            httpGet:
              path: /health
              port: 5001
            periodSeconds: 10
            failureThreshold: 30

          livenessProbe:
            httpGet:
              path: /health
              port: 5001
            initialDelaySeconds: 30
            periodSeconds: 10
            failureThreshold: 3

          readinessProbe:
            httpGet:
              path: /health
              port: 5001
            initialDelaySeconds: 5
            periodSeconds: 5
            failureThreshold: 3

          volumeMounts:
            - name: secrets-store
              mountPath: /mnt/secrets-store
              readOnly: true

      volumes:
        - name: secrets-store
          csi:
            driver: secrets-store.csi.k8s.io
            readOnly: true
            volumeAttributes:
              secretProviderClass: backend-secret-provider
```

## Explanation

### Deployment and namespace

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: backend-deployment
  namespace: chatapp
```

Creates a Deployment named `backend-deployment` in the `chatapp` namespace.

### Replicas

```yaml
replicas: 2
```

Maintains two backend pods for availability.

### Selector and labels

```yaml
selector:
  matchLabels:
    app: backend
```

and:

```yaml
labels:
  app: backend
```

The Deployment manages pods with the `app=backend` label. The backend Service uses the same label to route traffic to these pods.

### ServiceAccount

```yaml
serviceAccountName: backend-sa
```

Runs the backend pods using `backend-sa`, which is associated with the AWS IAM role through EKS Pod Identity.

### ECR image

```yaml
image: 236209348145.dkr.ecr.ap-southeast-1.amazonaws.com/chat-app-backend:7
```

Pulls backend image tag `7` from Amazon ECR in `ap-southeast-1`.

### Container port

```yaml
containerPort: 5001
```

The backend application listens on port `5001`.

### Environment variables

`MONGODB_URI` and `JWT_SECRET` are read from `backend-secret` using `secretKeyRef`. They are not hard-coded in the Deployment.

`PORT=5001` defines the application port and `NODE_ENV=production` runs the backend in production mode.

### Resources

```yaml
requests:
  cpu: "100m"
  memory: "256Mi"
limits:
  cpu: "500m"
  memory: "512Mi"
```

Requests are used for scheduling. Limits cap the resources the container can consume. These values also provide the baseline used by the HPA's utilization calculations.

### Startup probe

Checks `GET /health` on port `5001` during application startup. The application gets up to 30 failed checks at 10-second intervals before startup is considered unsuccessful.

### Liveness probe

Checks `GET /health` every 10 seconds after an initial 30-second delay. Repeated failures allow Kubernetes to restart an unhealthy container.

### Readiness probe

Checks `GET /health` every 5 seconds after an initial 5-second delay. A pod that is not ready does not receive Service traffic.

### Secrets Store CSI volume

```yaml
volumeMounts:
  - name: secrets-store
    mountPath: /mnt/secrets-store
    readOnly: true
```

Mounts the Secrets Store CSI volume as read-only.

The volume references:

```yaml
secretProviderClass: backend-secret-provider
```

which connects the pod to the AWS Secrets Manager integration defined in `sectes.yaml`.

## Request and secret flow

```text
ALB
 ↓
backend-service:5001
 ↓
Backend Pods
 ↓
backend-secret
 ↑
SecretProviderClass
 ↑
Secrets Store CSI Driver
 ↑
AWS Secrets Manager
```

## Verify

```bash
kubectl get deployment backend-deployment -n chatapp
kubectl get pods -n chatapp -l app=backend
kubectl describe deployment backend-deployment -n chatapp
```
