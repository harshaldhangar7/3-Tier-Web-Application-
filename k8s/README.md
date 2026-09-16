# Production Kubernetes Deployment Guide (Kubeadm)

This directory contains production-ready Kubernetes manifests designed for a self-hosted Kubernetes cluster initialized with **Kubeadm**.

---

## 📁 Manifest Structure

| File | Resource | Description |
| :--- | :--- | :--- |
| `00-namespace.yaml` | `Namespace` | Creates isolated `three-tier` namespace |
| `01-secret.yaml` | `Secret` | MySQL root & application passwords |
| `02-configmap.yaml` | `ConfigMap` | Database host/port/name & `init.sql` schema |
| `03-mysql-pv-pvc.yaml` | `PV` & `PVC` | Persistent disk storage for MySQL database |
| `04-mysql-statefulset.yaml` | `StatefulSet` + `Service` | MySQL 8.4 database with health probes & resource limits |
| `05-backend-deployment.yaml` | `Deployment` + `Service` | Flask backend (2 replicas) with `/health` probes |
| `06-frontend-deployment.yaml` | `Deployment` + `Service` | React/Vite frontend (2 replicas) served via NGINX |
| `07-ingress.yaml` | `Ingress` | Ingress-nginx routing (`/api`, `/health`, `/`) |
| `08-proxy-nodeport.yaml` | `Deployment` + `Service` | *Optional:* Standalone NGINX proxy on NodePort `30080` |
| `09-network-policy.yaml` | `NetworkPolicy` | Zero-trust pod-to-pod microsegmentation |
| `kustomization.yaml` | `Kustomization` | Unified deployment with Kustomize |

---

## 🏗️ Step 1: Build & Push Container Images

Because Kubeadm nodes need access to your container images, you must build and make them available to your cluster.

### Option A: Push to Docker Hub or Container Registry (Recommended)

```bash
# Set your registry username
export REGISTRY="your-dockerhub-username"

# 1. Build images
docker build -t $REGISTRY/three-tier-backend:1.0.0 ./backend
docker build -t $REGISTRY/three-tier-frontend:1.0.0 ./frontend

# 2. Push images
docker push $REGISTRY/three-tier-backend:1.0.0
docker push $REGISTRY/three-tier-frontend:1.0.0
```

> **Note:** Update the `image:` fields in `05-backend-deployment.yaml` and `06-frontend-deployment.yaml` (or uncomment the `images:` section in `kustomization.yaml`) with your registry prefix.

### Option B: Direct Import into Containerd (Single-Node / Local Kubeadm)

If testing on a single Kubeadm control-plane/worker without pushing to a registry:

```bash
# Build locally
docker build -t three-tier-backend:1.0.0 ./backend
docker build -t three-tier-frontend:1.0.0 ./frontend

# Save to tar and import into containerd k8s.io namespace
docker save three-tier-backend:1.0.0 -o backend.tar
docker save three-tier-frontend:1.0.0 -o frontend.tar

sudo ctr -n k8s.io images import backend.tar
sudo ctr -n k8s.io images import frontend.tar
```

---

## ⚙️ Step 2: Configure Storage & Secrets

1. **Review Credentials**:
   Edit `01-secret.yaml` and update the passwords:
   ```yaml
   stringData:
     MYSQL_ROOT_PASSWORD: "YourStrongRootPassword"
     MYSQL_USER: "app_user"
     MYSQL_PASSWORD: "YourStrongAppPassword"
   ```

2. **Storage for Kubeadm**:
   - If your cluster has a default dynamic StorageClass (e.g., `local-path-provisioner`, Longhorn, NFS), the PVC in `03-mysql-pv-pvc.yaml` will bind automatically.
   - If you do not have dynamic provisioning, create `/mnt/data/mysql` on your worker node and ensure the `mysql-pv` resource in `03-mysql-pv-pvc.yaml` is enabled.

---

## 🚀 Step 3: Deploy to Kubernetes

### Using Kustomize (All-In-One):

```bash
kubectl apply -k k8s/
```

### Or Applying Sequentially:

```bash
kubectl apply -f k8s/00-namespace.yaml
kubectl apply -f k8s/01-secret.yaml
kubectl apply -f k8s/02-configmap.yaml
kubectl apply -f k8s/03-mysql-pv-pvc.yaml
kubectl apply -f k8s/04-mysql-statefulset.yaml
kubectl apply -f k8s/05-backend-deployment.yaml
kubectl apply -f k8s/06-frontend-deployment.yaml
kubectl apply -f k8s/07-ingress.yaml
kubectl apply -f k8s/09-network-policy.yaml
```

---

## 🌐 Step 4: Access the Application

### Method 1: Via Ingress Controller (Production)
If you have `ingress-nginx` installed on your Kubeadm cluster:
- The Ingress in `07-ingress.yaml` routes:
  - `http://<Cluster-IP-or-Domain>/api/*` ➔ Backend
  - `http://<Cluster-IP-or-Domain>/health` ➔ Backend
  - `http://<Cluster-IP-or-Domain>/` ➔ Frontend

### Method 2: Via Standalone NodePort Proxy (Alternative for Bare-Metal)
If you do not have an Ingress controller configured, deploy the standalone proxy:
```bash
kubectl apply -f k8s/08-proxy-nodeport.yaml
```
Access the application directly in your browser:
```text
http://<Any-Worker-Node-IP>:30080
```

---

## 🔍 Verification & Troubleshooting

### Check Pod Status:
```bash
kubectl get pods -n three-tier -o wide
```

Expected output:
```text
NAME                        READY   STATUS    RESTARTS   AGE
backend-xxxxxxxxxx-xxxxx    1/1     Running   0          2m
backend-xxxxxxxxxx-xxxxx    1/1     Running   0          2m
frontend-xxxxxxxxxx-xxxxx   1/1     Running   0          2m
frontend-xxxxxxxxxx-xxxxx   1/1     Running   0          2m
mysql-0                     1/1     Running   0          3m
```

### Check Logs:
```bash
# Backend logs:
kubectl logs -n three-tier -l app=backend -f

# MySQL logs:
kubectl logs -n three-tier mysql-0 -f
```

### Check Persistent Volume:
```bash
kubectl get pvc,pv -n three-tier
```

### Testing Internal Connectivity:
```bash
# Verify backend can query MySQL:
kubectl exec -it -n three-tier deploy/backend -- curl -s http://localhost:5000/health
# Response: {"database":"connected","service":"flask-backend","status":"ok"}
```
