# ☸️ Kubernetes Quick Reference

## Core Concepts

| Resource         | Description |
|------------------|-------------|
| Pod              | Smallest deployable unit; one or more containers |
| Node             | Worker machine (VM or bare metal) |
| Cluster          | Set of nodes managed by Kubernetes |
| Namespace        | Virtual cluster within a cluster |
| Deployment       | Manages pod replicas and rolling updates |
| Service          | Stable network endpoint for pods |
| ConfigMap        | Non-sensitive configuration data |
| Secret           | Sensitive data (passwords, keys) |
| PersistentVolume | Durable storage resource |
| Ingress          | HTTP/HTTPS routing into the cluster |

## kubectl Cheat Sheet

```bash
# Cluster info
kubectl cluster-info
kubectl get nodes
kubectl get nodes -o wide

# Namespace
kubectl get namespaces
kubectl create namespace myns
kubectl config set-context --current --namespace=myns

# Apply / Delete resources
kubectl apply  -f manifest.yaml
kubectl delete -f manifest.yaml
kubectl apply  -f ./k8s/           # directory

# Get resources
kubectl get pods
kubectl get pods -n myns
kubectl get pods --all-namespaces
kubectl get deployments
kubectl get services
kubectl get all

# Describe (detailed info + events)
kubectl describe pod mypod
kubectl describe deployment myapp

# Logs
kubectl logs mypod
kubectl logs mypod -c mycontainer  # specific container
kubectl logs -f mypod              # follow
kubectl logs --previous mypod      # crashed container

# Shell into pod
kubectl exec -it mypod -- bash
kubectl exec -it mypod -c mycontainer -- sh

# Port forward
kubectl port-forward pod/mypod 8080:80
kubectl port-forward svc/myservice 8080:80

# Scale
kubectl scale deployment myapp --replicas=5

# Rolling update
kubectl set image deployment/myapp mycontainer=myimage:2.0
kubectl rollout status deployment/myapp
kubectl rollout history deployment/myapp
kubectl rollout undo deployment/myapp           # rollback
kubectl rollout undo deployment/myapp --to-revision=2

# Copy files
kubectl cp mypod:/path/to/file ./local/file
kubectl cp ./local/file mypod:/path/to/file
```

## Common Manifests

### Pod

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: my-pod
  labels:
    app: my-app
spec:
  containers:
    - name: my-container
      image: nginx:1.25
      ports:
        - containerPort: 80
      resources:
        requests:
          memory: "64Mi"
          cpu: "250m"
        limits:
          memory: "128Mi"
          cpu: "500m"
      livenessProbe:
        httpGet:
          path: /health
          port: 80
        initialDelaySeconds: 10
        periodSeconds: 5
      readinessProbe:
        httpGet:
          path: /ready
          port: 80
        initialDelaySeconds: 5
        periodSeconds: 3
```

### Deployment

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: my-app
  namespace: production
spec:
  replicas: 3
  selector:
    matchLabels:
      app: my-app
  strategy:
    type: RollingUpdate
    rollingUpdate:
      maxSurge: 1
      maxUnavailable: 0
  template:
    metadata:
      labels:
        app: my-app
        version: "2.0"
    spec:
      containers:
        - name: my-app
          image: my-registry/my-app:2.0
          ports:
            - containerPort: 3000
          env:
            - name: NODE_ENV
              value: production
            - name: DB_PASSWORD
              valueFrom:
                secretKeyRef:
                  name: db-secret
                  key: password
          envFrom:
            - configMapRef:
                name: app-config
          resources:
            requests:
              cpu: "100m"
              memory: "128Mi"
            limits:
              cpu: "500m"
              memory: "512Mi"
```

### Service

```yaml
# ClusterIP (internal only)
apiVersion: v1
kind: Service
metadata:
  name: my-service
spec:
  selector:
    app: my-app
  ports:
    - protocol: TCP
      port: 80
      targetPort: 3000
  type: ClusterIP

---
# NodePort (expose on node port)
spec:
  type: NodePort
  ports:
    - port: 80
      targetPort: 3000
      nodePort: 30080   # 30000-32767

---
# LoadBalancer (cloud provider LB)
spec:
  type: LoadBalancer
  ports:
    - port: 80
      targetPort: 3000
```

### ConfigMap & Secret

```yaml
# ConfigMap
apiVersion: v1
kind: ConfigMap
metadata:
  name: app-config
data:
  APP_PORT: "3000"
  LOG_LEVEL: "info"
  config.json: |
    {
      "feature_flags": { "dark_mode": true }
    }

---
# Secret (values must be base64 encoded)
apiVersion: v1
kind: Secret
metadata:
  name: db-secret
type: Opaque
data:
  password: cGFzc3dvcmQxMjM=   # base64: password123
  username: YWRtaW4=            # base64: admin
```

### Ingress

```yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: my-ingress
  annotations:
    nginx.ingress.kubernetes.io/rewrite-target: /
spec:
  ingressClassName: nginx
  tls:
    - hosts:
        - myapp.example.com
      secretName: tls-secret
  rules:
    - host: myapp.example.com
      http:
        paths:
          - path: /api
            pathType: Prefix
            backend:
              service:
                name: api-service
                port:
                  number: 80
          - path: /
            pathType: Prefix
            backend:
              service:
                name: frontend-service
                port:
                  number: 80
```

### HorizontalPodAutoscaler

```yaml
apiVersion: autoscaling/v2
kind: HorizontalPodAutoscaler
metadata:
  name: my-app-hpa
spec:
  scaleTargetRef:
    apiVersion: apps/v1
    kind: Deployment
    name: my-app
  minReplicas: 2
  maxReplicas: 10
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

## Key Patterns

```
Pod design:
- One process per container (single responsibility)
- Use init containers for setup tasks
- Use sidecar containers for logging, proxies

Resource management:
- Always set resource requests AND limits
- requests = scheduling guarantee
- limits = hard cap (CPU throttled, memory OOMKilled)

Health checks:
- livenessProbe:  restart container if unhealthy
- readinessProbe: remove from service endpoints if not ready
- startupProbe:   disable liveness during slow startup

Labels & selectors:
- app, version, environment, tier (common label conventions)
- Used by Services, Deployments, HPA to select pods
```
