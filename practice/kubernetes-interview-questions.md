# Kubernetes Interview Questions

**1. What is Kubernetes and why do we need it?**
Kubernetes is a container orchestration platform. It automates deployment, scaling, self-healing, and networking of containerized applications across clusters of machines. Needed because manually managing hundreds of containers across many servers is impractical.

**2. What is a Pod?**
The smallest deployable unit in Kubernetes. One or more containers sharing a network namespace and storage volumes. Containers in a Pod communicate via `localhost`.

**3. What is the difference between a Pod and a Deployment?**
A Pod is a single instance. A Deployment manages a set of identical Pods via a ReplicaSet, ensuring the desired number always runs, and handles rolling updates/rollbacks.

**4. What are Service types in Kubernetes?**
- `ClusterIP` (default): Internal only. `NodePort`: Exposes on each node's IP. `LoadBalancer`: Provisions a cloud load balancer. `ExternalName`: DNS alias for external services.

**5. What is a Namespace?**
A way to logically partition cluster resources. Useful for isolating environments (dev, staging, prod) or teams within the same cluster.

**6. What is a ConfigMap vs a Secret?**
Both decouple configuration from images. ConfigMaps store plain text config. Secrets store sensitive data (base64-encoded, with optional encryption at rest). Both can be mounted as env vars or files.

**7. What is the Kubernetes Control Plane?**
The master components: API Server (entry point), etcd (cluster state DB), Scheduler (assigns Pods to Nodes), Controller Manager (runs reconciliation loops).

**8. What is a ReplicaSet?**
Ensures a specified number of Pod replicas are running at all times. Normally managed by a Deployment — you rarely create ReplicaSets directly.

**9. How does Kubernetes self-heal?**
The Controller Manager runs a reconciliation loop comparing desired state (e.g., 3 replicas) vs actual state. If a Pod crashes, it creates a new one. If a node fails, its pods are rescheduled on healthy nodes.

**10. What is a rolling update vs a recreate strategy?**
- **RollingUpdate**: Replaces Pods incrementally. Zero downtime. Configurable via `maxSurge` and `maxUnavailable`.
- **Recreate**: Kills all old Pods then creates new ones. Causes downtime but avoids running two versions simultaneously.

**11. What is Ingress?**
An API object that manages external HTTP/HTTPS access to services. A single Ingress controller (nginx, Traefik) can route traffic to multiple services based on hostname or URL path, replacing the need for one LoadBalancer per service.

**12. What is Helm?**
Kubernetes package manager. Helm Charts bundle all manifests for an application with configurable values. Enables versioned installs, upgrades, and rollbacks of complex applications.

**13. What is the difference between `kubectl apply` and `kubectl create`?**
`kubectl create` fails if the resource already exists. `kubectl apply` creates if not exists or patches if it does (declarative, idempotent). **Always use `apply` in CI/CD.**

**14. What are resource requests and limits?**
`requests` = guaranteed minimum resources (CPU, memory) the scheduler uses to place the Pod. `limits` = maximum. Exceeding CPU limit throttles the container. Exceeding memory limit kills the container (OOMKilled).
