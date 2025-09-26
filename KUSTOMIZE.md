# Kustomize Deployment Guide

This guide explains how to deploy the DumbKV application using Kustomize.

## Quick Start

### Prerequisites

- Kubernetes cluster
- `kubectl` configured to access your cluster
- NGINX Ingress Controller (for external access)

### Deploy Everything

**Single command deployment:**
```bash
kubectl apply -k kustomize/base/
```

The entire application stack will be deployed including:
- cert-manager (for SSL certificates)
- PostgreSQL database
- DumbKV application
- SSL certificate management

### To delete everything

**Single command cleanup:**
```bash
kubectl delete -k kustomize/base/
```

## 🔍 Verification

Check that everything is running:
```bash
# Check all pods
kubectl get pods

# Check services
kubectl get services

# Check ingress
kubectl get ingress

# Check SSL certificate status
kubectl get clusterissuer
kubectl describe clusterissuer letsencrypt
```

## 🌐 Accessing the Application

### Local Development

If using a local cluster (like Docker Desktop or minikube):

1. Get the ingress IP:
```bash
kubectl get ingress dumbkv-ingress
```

2. Add to your `/etc/hosts` file:
```
<INGRESS_IP> dumbkv.example.com
```

3. Access the application:
- HTTP: `http://dumbkv.example.com`
- HTTPS: `https://dumbkv.example.com` (certificate will show as unsafe due to fake domain)

## 🔧 Configuration

### Environment Variables

The application uses these environment variables:
- `DATABASE_TYPE`: Set to "postgres"
- `DATABASE_LOCATION`: PostgreSQL connection string from secret

### Resource Limits

- **CPU**: 100m request, 500m limit
- **Memory**: 128Mi request, 512Mi limit

### Health Checks

- **Liveness Probe**: `/health` endpoint, 30s initial delay
- **Readiness Probe**: `/health` endpoint, 5s initial delay

## 🐛 Troubleshooting

### Check Application Logs
```bash
kubectl logs -l app.kubernetes.io/name=dumbkv
```

### Check Database Connection
```bash
kubectl logs -l app.kubernetes.io/name=dumbkv | grep -i postgres
```

### Check SSL Certificate Status
```bash
kubectl describe certificate dumbkv-tls
```

### Check Ingress Status
```bash
kubectl describe ingress dumbkv-ingress
```

