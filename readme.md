📘 DevSecOps Multi-Tenant Monitoring (Kubernetes + Grafana)
🎯 Objective

Build a secure system where:
```
1 User = 1 Namespace = 1 Dashboard
```
Each user:
    can deploy only in their namespace
    can use only their PVC
    can monitor only their own resources
    cannot see other users or system dashboards

🏗️ Architecture
```
User (devith)
   ↓
Kubernetes Namespace (ns-devith)
   ↓
RBAC (restrict access)
   ↓
PVC (Longhorn storage)
   ↓
Prometheus (collect metrics)
   ↓
Grafana (filtered dashboard)
   ↓
User sees only own dashboard
```
🚀 Step 1 — Create Namespace
```YAML
apiVersion: v1
kind: Namespace
metadata:
  name: ns-devith
```
```bash
kubectl apply -f namespace.yaml
```

🔐 Step 2 — Create User (Certificate)
```bash
# generate key
openssl genrsa -out devith.key 2048

# create CSR
openssl req -new -key devith.key -out devith.csr -subj "/CN=devith/O=dev"

# sign certificate
sudo openssl x509 -req -in devith.csr \
  -CA /etc/kubernetes/pki/ca.crt \
  -CAkey /etc/kubernetes/pki/ca.key \
  -CAcreateserial \
  -out devith.crt \
  -days 365
```

⚙️ Step 3 — Configure kubeconfig
```bash
kubectl config set-credentials devith \
  --client-certificate=/home/enz/devith.crt \
  --client-key=/home/enz/devith.key

kubectl config set-context devith-context \
  --cluster=benzcluster \
  --user=devith \
  --namespace=ns-devith
```
```bash
# switch user

kubectl config use-context kubernetes-admin@benzcluster

kubectl config use-context devith-context
```
🛑 Step 4 — Apply RBAC

💾 Step 5 — Storage (PVC with Longhorn)

📦 Step 6 — ResourceQuota (Limit Storage)

🧪 Step 7 — Test Pod

📊 Step 8 — Monitoring Setup

Ensure:
    Prometheus is running
    kube-state-metrics is installed
    Metrics include namespace="ns-devith"

Test:
```bash
kube_pod_info{namespace="ns-devith"}
```

📈 Step 9 — Create Grafana Dashboard

Create new dashboard:
```bash
#Pod Count
count(kube_pod_info{namespace="ns-devith"})

#CPU Usage
sum(rate(container_cpu_usage_seconds_total{namespace="ns-devith"}[5m])) by (pod)

#Memory Usage
sum(container_memory_working_set_bytes{namespace="ns-devith"}) by (pod)

#Storage Usage
sum(kube_persistentvolumeclaim_resource_requests_storage_bytes{namespace="ns-devith"})
```
Save as:
tenant-ns-devith

📁 Step 10 — Grafana Folder Isolation

Create folder:
```
devith
```
Move dashboard into it.

🔒 Step 11 — Create Grafana User
```
Username: devith
Role: Viewer
```

🔐 Step 12 — Set Folder Permissions
devith folder

User: devith → View
Remove:
    Viewer ❌
    Editor ❌