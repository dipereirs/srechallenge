# Kubernetes Manifests

This directory contains Kubernetes manifests for deploying the DumbKV application.
This part was done using the MacOs Docker Kubernets

## Prerequisites

### 1. NGINX Ingress Controller
```bash
kubectl apply -f https://raw.githubusercontent.com/kubernetes/ingress-nginx/controller-v1.8.2/deploy/static/provider/cloud/deploy.yaml
```

Verify if everything is running correctly

```bash
kubectl get pods -n ingress-nginx
kubectl logs nginx-pod-name -n ingress-nginx
```

### 2. cert-manager
```bash
# Install cert-manager CRDs (Had to do this because using helm it was hanging)
kubectl apply -f https://github.com/cert-manager/cert-manager/releases/download/v1.18.2/cert-manager.crds.yaml

# Install cert-manager using Helm - this can take a little time
helm repo add jetstack https://charts.jetstack.io
helm repo update
helm install cert-manager jetstack/cert-manager \
  --namespace cert-manager \
  --create-namespace \
  --version v1.18.2 \
  --set installCRDs=false
```

Verify if everything is running correctly

```bash
# Check cert-manager
kubectl get pods -n cert-manager
# Check logs for any error or if it's restarting
kubectl logs pods-found -n cert-manager
```

### 3. Prometheus & Grafana (Monitoring Stack)
```bash
# Add Prometheus Helm repository
helm repo add prometheus-community https://prometheus-community.github.io/helm-charts
helm repo update

# Install Prometheus, Grafana, and AlertManager
helm install prometheus prometheus-community/kube-prometheus-stack
```

Verify if everything is working correctly

```bash
# Check cert-manager
kubectl get pods | grep prometheus
# Check logs for any error or if it's restarting
kubectl logs pods-found
```

## Deployment steps

Deploy in the following order

### Apply the PostgreSQL secret and start the service:
Access the manifests folder and run all the following:

```bash
kubectl apply -f postgressql-secret.yaml
#check if secret is created
kubectl get secrets | grep postgres
# this starts a PVC, deployment and a service
kubectl apply -f postgresql.yaml
# to check pvc, deployment and service we can run
kubectl get pvc
kubectl get deployment
kubectl get pods | grep postgres
kubectl logs postgres-pod
```

### Apply the DumbKV application:
```bash
kubectl apply -f dumbkv-service.yaml
# check deployment and pods
kubect get deployment | grep dumbkv
kubect get pods | grep dumbkv
kubectl logs dumbkvpod
```

### Apply the ClusterIssuer to generate the https certificate :
```bash
kubectl apply -f cluster-issuer.yaml
```

Verify the ClusterIssuer is working:
```bash
# Check if ClusterIssuer was created
kubectl get clusterissuer
# Check ClusterIssuer status and conditions
kubectl describe clusterissuer letsencrypt
```

### Apply the ingress:
```bash
kubectl apply -f ingress.yaml
```

Verify the ingress and certificate:
```bash
# Check if ingress was created
kubectl get ingress
# Check ingress details and certificate status
kubectl describe ingress dumbkv-ingress
# Check if certificate was created
kubectl get certificate
# Check certificate status and events
# this is where I couldn't bypass Warning  Failed     26s   cert-manager-certificates-issuing          The certificate request has failed to complete and will be retried: Failed to wait for order resource "dumbkv-tls-1-1953608086" to become ready: order is in "errored" state: Failed to create Order: 400 urn:ietf:params:acme:error:rejectedIdentifier: Error creating new order :: Cannot issue for "dumbkv.example.com": The ACME server refuses to issue a certificate for this domain name, because it is forbidden by policy
kubectl describe certificate dumbkv-tls
# Check certificate order and status
kubectl get certificaterequest
kubectl get order
```

### Local Validation
```bash
kubectl get ingress
```
Add the ingress address to the etc hosts like 0.0.0.0 dumbkv-example.com as sudo and save it
then access https://dumbkv.example.com/, it will give an error saying the site isn't safe because of the certificate issue stated before
using chrome you can type thisisunsafe and the page will show

### Local prometheus and grafana access

Access Prometheus:
```bash
# Port forward to Prometheus
kubectl port-forward service/prometheus-kube-prometheus-prometheus 9090:9090
```
Then access: http://localhost:9090

Access Grafana:
```bash
# Get Grafana admin password
kubectl get secret prometheus-grafana -o jsonpath="{.data.admin-password}" | base64 --decode

# Port forward to Grafana
kubectl port-forward service/prometheus-grafana 3000:80
```
Then access: http://localhost:3000
- Username: `admin`
- Password: (use the decoded password from above command)


## Notes
- The ClusterIssuer is configured for Let's Encrypt staging environment
- Certificates will fail to issue due to fake domain, certificate will show as unsafe (maybe I did something wrong)
- ServiceMonitor was failing due to the `release: prometheus` label in prometheus so I had to add it