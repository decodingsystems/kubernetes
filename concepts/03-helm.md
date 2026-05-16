# Helm — Kubernetes Package Manager

## What is Helm?
**Helm** is the package manager for Kubernetes. Just as `apt` makes installing Linux packages easy, Helm makes deploying complex Kubernetes applications easy via reusable, versioned packages called **Charts**.

A Chart bundles all the YAML manifests (Deployments, Services, ConfigMaps, Ingress, etc.) needed for an application into a single, parameterized, versioned artifact.

---

## Key Concepts

### Chart
A directory/archive with a standard structure:
```
my-app/
  Chart.yaml          # Chart metadata (name, version, description)
  values.yaml         # Default configuration values
  templates/          # Go-templated Kubernetes YAML manifests
    deployment.yaml
    service.yaml
    ingress.yaml
  charts/             # Chart dependencies (sub-charts)
```

### Templates and Values
Templates use Go templating syntax to inject values from `values.yaml`:
```yaml
# templates/deployment.yaml
spec:
  replicas: {{ .Values.replicaCount }}
  template:
    spec:
      containers:
      - name: {{ .Chart.Name }}
        image: "{{ .Values.image.repository }}:{{ .Values.image.tag }}"
```

```yaml
# values.yaml
replicaCount: 3
image:
  repository: nginx
  tag: "1.25"
```

### Release
When you install a Chart, Helm creates a **Release** — a named instance of the chart in the cluster. You can have multiple releases of the same chart (e.g., `my-app-prod`, `my-app-staging`).

---

## Essential Helm Commands

```bash
# Add a chart repository
helm repo add bitnami https://charts.bitnami.com/bitnami
helm repo update                         # Refresh repo index

# Search for charts
helm search repo bitnami/postgresql

# Install a chart (creates a release)
helm install my-db bitnami/postgresql

# Install with custom values
helm install my-db bitnami/postgresql \
  --set auth.postgresPassword=mysecret \
  --set primary.persistence.size=10Gi

# Install from a values file
helm install my-db bitnami/postgresql -f my-values.yaml

# List releases
helm list

# Upgrade a release
helm upgrade my-db bitnami/postgresql --set image.tag=16.1.0

# Rollback to previous release version
helm rollback my-db 1

# View release history
helm history my-db

# Uninstall a release
helm uninstall my-db
```

---

## Why Use Helm?

| Without Helm | With Helm |
|-------------|-----------|
| Manage 20 separate YAML files | One `helm install` command |
| Copy-paste for dev/staging/prod | Different `values.yaml` per environment |
| No version history | `helm history` + `helm rollback` |
| Manual database setup | `helm install bitnami/postgresql` |

---

## Hands-on Lab: Deploy Nginx with Helm

### Prerequisites
```bash
# Install Helm (Windows with Chocolatey)
choco install kubernetes-helm
# Or Linux
curl https://raw.githubusercontent.com/helm/helm/main/scripts/get-helm-3 | bash
```

### Steps
1. **Start Minikube and add a repo:**
   ```bash
   minikube start
   helm repo add bitnami https://charts.bitnami.com/bitnami
   helm repo update
   ```
2. **Inspect the chart before installing:**
   ```bash
   helm show values bitnami/nginx | head -50   # See all defaults
   ```
3. **Install with custom values:**
   ```bash
   helm install my-nginx bitnami/nginx \
     --set service.type=NodePort \
     --set replicaCount=2
   ```
4. **Verify:**
   ```bash
   helm list
   kubectl get pods        # 2 nginx pods
   kubectl get service     # NodePort service
   minikube service my-nginx --url
   ```
5. **Upgrade to 3 replicas:**
   ```bash
   helm upgrade my-nginx bitnami/nginx --set replicaCount=3 --set service.type=NodePort
   kubectl get pods        # 3 pods now
   helm history my-nginx   # Shows revision 1 and 2
   ```
6. **Rollback:**
   ```bash
   helm rollback my-nginx 1
   kubectl get pods        # Back to 2 pods
   ```
7. **Cleanup:**
   ```bash
   helm uninstall my-nginx
   minikube stop
   ```
