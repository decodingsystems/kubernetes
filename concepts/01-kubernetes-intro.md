# Kubernetes — Container Orchestration

## What is Kubernetes?
**Kubernetes** (K8s) is an open-source container orchestration platform. While Docker runs individual containers, Kubernetes manages **clusters of containers** at scale — handling deployment, scaling, self-healing, and networking across fleets of machines.

## Kubernetes Architecture

```
┌─────────────── Control Plane (Master) ─────────────────┐
│   API Server  │  Scheduler  │  etcd  │  Controller Mgr  │
└────────────────────────────────────────────────────────┘
                         │
         ┌───────────────┼───────────────┐
         ▼               ▼               ▼
     Worker Node     Worker Node     Worker Node
   ┌────────────┐
   │  kubelet   │  ← Agent on every node
   │  kube-proxy│  ← Manages network rules
   │  container │
   │  runtime   │
   └────────────┘
```

- **API Server:** Front-end of K8s. All `kubectl` commands go here.
- **etcd:** Distributed key-value store — the cluster's source of truth.
- **Scheduler:** Decides which node a new Pod should run on.
- **Controller Manager:** Reconciliation loops maintaining desired state.
- **kubelet:** Agent on every worker node ensuring containers run as specified.

## Core Objects

### Pod
Smallest deployable unit. One or more containers sharing a network namespace and storage. Pods are ephemeral — never create them directly; use a Deployment instead.

### Deployment
Manages a ReplicaSet of identical Pods. Handles rolling updates and rollbacks.
```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: web-app
spec:
  replicas: 3
  selector:
    matchLabels:
      app: web-app
  template:
    metadata:
      labels:
        app: web-app
    spec:
      containers:
      - name: web
        image: nginx:1.25
        ports:
        - containerPort: 80
```

### Service
Stable network endpoint (IP + DNS) that load-balances traffic to Pods matched by labels.
```yaml
apiVersion: v1
kind: Service
spec:
  selector:
    app: web-app
  ports:
  - port: 80
    targetPort: 80
  type: ClusterIP    # ClusterIP | NodePort | LoadBalancer
```

### ConfigMap and Secret
- **ConfigMap:** Non-sensitive config (env vars, config files).
- **Secret:** Sensitive data (passwords, API keys) — base64 encoded.

## kubectl Essentials
```bash
kubectl get pods                    # List pods
kubectl describe pod <name>         # Detailed info + events
kubectl logs <pod-name>             # View pod logs
kubectl exec -it <pod-name> -- bash # Shell into a pod
kubectl apply -f deployment.yaml    # Apply manifest
kubectl scale deployment web-app --replicas=5
kubectl rollout undo deployment/web-app   # Rollback
```
