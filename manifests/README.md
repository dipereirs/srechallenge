# Kubernetes Manifests

This directory contains Kubernetes manifests for deploying the DumbKV application.
This part was done using the MacOs Docker Kubernets

## Prerequisites

### 1. NGINX Ingress Controller
```bash
kubectl apply -f https://raw.githubusercontent.com/kubernetes/ingress-nginx/controller-v1.8.2/deploy/static/provider/cloud/deploy.yaml
```

### 2. cert-manager
```bash
# Install cert-manager CRDs (Had to do this because using helm it was hanging)
kubectl apply -f https://github.com/cert-manager/cert-manager/releases/download/v1.18.2/cert-manager.crds.yaml

# Install cert-manager using Helm
helm repo add jetstack https://charts.jetstack.io
helm repo update
helm install cert-manager jetstack/cert-manager \
  --namespace cert-manager \
  --create-namespace \
  --version v1.18.2 \
  --set installCRDs=false
```

### 3. Verify Installation
```bash
# Check NGINX Ingress Controller
kubectl get pods -n ingress-nginx

# Check cert-manager
kubectl get pods -n cert-manager
```

## Deployment

Deploy in the following order

1. Apply the PostgreSQL secret:
```bash
kubectl apply -f postgressql-secret.yaml
```

2. Apply PostgreSQL database:
```bash
kubectl apply -f postgresql.yaml
```

3. Wait for PostgreSQL to be ready:
```bash
kubectl wait --for=condition=ready pod -l app.kubernetes.io/name=postgres --timeout=120s
```

4. Apply the DumbKV application:
```bash
kubectl apply -f dumbkv-service.yaml
```

5. Apply the ClusterIssuer:
```bash
kubectl apply -f cluster-issuer.yaml
```

6. Apply the ingress:
```bash
kubectl apply -f ingress.yaml
```

## Notes

- The ClusterIssuer is configured for Let's Encrypt staging environment
- Certificates will fail to issue due to fake domain, certificate will show as unsafe (maybe I did something wrong)