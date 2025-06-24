# 🚀 EKS + Datadog Integration

This project demonstrates how to integrate Datadog with an EKS cluster, enabling full observability including metrics, logs, and APM. It also includes a working 404 error alert based on NGINX logs for the `project04` deployment.

---

## ⚙️ What’s Included

- Datadog Agent + Cluster Agent setup via Helm
- Log collection from Kubernetes containers
- Deployment annotation for log parsing
- Log monitor for HTTP 404 error detection
- GitOps-ready YAML structure for repeatability

---

## 🔧 Commands to Apply Setup

> Replace `<YOUR_DATADOG_API_KEY>` with your actual key.

```bash
# Create EKS Cluster
eksctl create cluster \
  --name datadog-eks \
  --region ap-south-1 \
  --nodegroup-name datadog \
  --node-type t3.small \
  --managed \
  --nodes 4

# Add Helm repo & install Datadog
helm repo add datadog https://helm.datadoghq.com
helm repo update

# Create secret with your Datadog API key
kubectl create secret generic datadog-secret \
  --from-literal api-key=<YOUR_DATADOG_API_KEY> \
  --namespace default

# Install Datadog Agent using repo-hosted values.yaml
helm install datadog-agent datadog/datadog \
  -f https://raw.githubusercontent.com/Sharathchandra3/Eks-Datadog-Integration/main/datadog-values.yaml \
  --namespace default

# Download & apply patch for log annotation
Invoke-WebRequest -Uri "https://raw.githubusercontent.com/Sharathchandra3/Eks-Datadog-Integration/main/datadog-log-patch.yaml" -OutFile "datadog-log-patch.yaml"
kubectl patch deployment project04-deployment --type merge --patch-file datadog-log-patch.yaml

# Verify everything is running
kubectl get pods
kubectl get deployments

🛎️ Create 404 Error Log Monitor in Datadog

Search Query:
service:project04 source:nginx " 404 "

Alert condition:
Alert when log count is above 10 over the last 5 minutes

Notification Message:

🚨 [NGINX 404 Error] Alert: {{value}} HTTP 404 errors detected in project04!

🔧 Filters:
- Service: `project04`
- Source: `nginx`

📍 Host: {{host.name}}  
🕒 Timestamp: {{timestamp}}

Please check for broken links, missing routes, or misconfigured frontend paths.

📧 Alert sent to: @user@gmail.com
Make sure @user@gmail.com is a registered user in your Datadog account to receive alerts.
