# AI Task Processing Platform (Infrastructure & GitOps)

This repository contains the Kubernetes manifests and Argo CD configuration for deploying the AI Task Processing Platform.

## Directory Structure
- `k8s/`: Contains all Kubernetes manifests (Deployments, Services, ConfigMaps, Secrets, Ingress, HPA).
- `argocd-app.yaml`: The Argo CD Application definition that points to this repository for GitOps auto-sync.
- `ARCHITECTURE.md`: Detailed architecture design and scalability documentation.

## Deployment Instructions

### Prerequisites
- A Kubernetes cluster (e.g., Minikube, k3d, Docker Desktop Kubernetes, AWS EKS).
- `kubectl` installed and configured.
- Argo CD installed on the cluster.

### 1. Apply Secrets Manually
Before syncing, ensure your secrets are applied. For local testing, you can apply the provided mock secret:
```bash
kubectl apply -f k8s/namespace.yaml
kubectl apply -f k8s/secret.yaml
```
*(In production, use external secrets management like HashiCorp Vault or AWS Secrets Manager).*

### 2. Deploy via Argo CD
1. Edit `argocd-app.yaml` and update the `repoURL` to point to your GitHub fork of this repository.
2. Apply the Argo CD application manifest:
```bash
kubectl apply -f argocd-app.yaml
```
3. Argo CD will automatically detect the manifests in the `k8s/` folder and sync them to your cluster.

### 3. Accessing the Application
Since we use an Ingress resource, ensure your cluster has an Ingress Controller (like Nginx Ingress).
- Map `localhost` or your cluster IP to access the frontend via port `80`.

## Scalability
The Backend and Worker deployments utilize Horizontal Pod Autoscalers (HPA) configured to scale between 2 and 10 replicas based on CPU load (70% utilization), allowing the system to easily handle up to 100,000 tasks/day. See `ARCHITECTURE.md` for more details.
