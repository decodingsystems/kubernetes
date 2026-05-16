# Kubernetes — Deployments, Services, and Networking

## Rolling Updates and Rollbacks

One of Kubernetes' most powerful features is **zero-downtime deployments** via rolling updates. Instead of taking down all pods and replacing them (a big-bang deploy), Kubernetes replaces pods one at a time.

```bash
# Update to a new image version
kubectl set image deployment/web-app web=nginx:1.26
kubectl rollout status deployment/web-app    # Watch it progress

# View rollout history
kubectl rollout history deployment/web-app

# Rollback to previous version if something went wrong
kubectl rollout undo deployment/web-app
kubectl rollout undo deployment/web-app --to-revision=2  # Specific revision
```

### Rollout Strategy Config
```yaml
spec:
  strategy:
    type: RollingUpdate
    rollingUpdate:
      maxSurge: 1         # Max extra pods above desired count during update
      maxUnavailable: 0   # Never take a pod down before a new one is up
```

---

## Service Types

| Type | Accessibility | Use Case |
|------|--------------|----------|
| `ClusterIP` | Internal only (default) | Service-to-service communication inside cluster |
| `NodePort` | External via `NodeIP:Port` | Dev/testing, direct node access |
| `LoadBalancer` | External via cloud load balancer | Production external traffic (AWS ELB, GCP LB) |
| `ExternalName` | DNS alias to external service | Connect to external DBs via K8s DNS |

---

## Ingress — HTTP Routing Into the Cluster
Instead of exposing each service with a `LoadBalancer` (expensive and one IP per service), use a single **Ingress** controller (nginx, Traefik) to route HTTP/HTTPS traffic to multiple services based on hostname or path.

```yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: my-ingress
spec:
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

---

## ConfigMaps and Secrets

### ConfigMap
```yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: app-config
data:
  DATABASE_HOST: "db.example.com"
  LOG_LEVEL: "info"
```

### Reference ConfigMap in a Pod
```yaml
envFrom:
- configMapRef:
    name: app-config
```

### Secret
```bash
# Create from CLI (values are base64 encoded automatically)
kubectl create secret generic db-secret \
  --from-literal=DB_PASSWORD=supersecret

# Or in YAML (values must be base64 encoded manually)
# echo -n "supersecret" | base64
```

---

## Resource Requests and Limits
```yaml
resources:
  requests:
    memory: "64Mi"    # Guaranteed minimum
    cpu: "250m"       # 250 millicores = 0.25 CPU
  limits:
    memory: "128Mi"   # Max before OOMKilled
    cpu: "500m"       # Max before throttled
```

---

## Hands-on Lab: Full App Deployment with ConfigMap and Rolling Update

### Steps
1. **Create a ConfigMap:**
   ```yaml
   # config.yaml
   apiVersion: v1
   kind: ConfigMap
   metadata:
     name: app-config
   data:
     APP_COLOR: "blue"
   ```
   ```bash
   kubectl apply -f config.yaml
   ```
2. **Create a Deployment using the ConfigMap:**
   ```yaml
   # deploy.yaml
   apiVersion: apps/v1
   kind: Deployment
   metadata:
     name: color-app
   spec:
     replicas: 3
     selector:
       matchLabels:
         app: color-app
     template:
       metadata:
         labels:
           app: color-app
       spec:
         containers:
         - name: app
           image: nginx:alpine
           env:
           - name: APP_COLOR
             valueFrom:
               configMapKeyRef:
                 name: app-config
                 key: APP_COLOR
   ```
   ```bash
   kubectl apply -f deploy.yaml
   kubectl exec -it <pod-name> -- env | grep APP_COLOR  # Verify env var
   ```
3. **Expose and test:**
   ```bash
   kubectl expose deployment color-app --type=NodePort --port=80
   minikube service color-app --url
   ```
4. **Perform a Rolling Update:**
   ```bash
   kubectl set image deployment/color-app app=nginx:1.25
   kubectl rollout status deployment/color-app
   # No downtime!
   ```
5. **Rollback:**
   ```bash
   kubectl rollout undo deployment/color-app
   kubectl rollout history deployment/color-app
   ```
6. **Cleanup:**
   ```bash
   kubectl delete deployment color-app
   kubectl delete service color-app
   kubectl delete configmap app-config
   ```
