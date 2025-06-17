# 🚀 Helm Install Guide

> This guide walks you through installing applications using Helm charts from public repositories like Bitnami.

---

## 📘 Step 01: Introduction

You will learn how to:

* Add Helm repositories
* Search for charts
* Install a chart
* View Helm releases
* Uninstall a release

**Commands Covered:**

* `helm repo list`
* `helm repo add`
* `helm repo update`
* `helm search repo`
* `helm install`
* `helm list`
* `helm uninstall`

---

## 📦 Step 02: Helm Repository Operations

🔗 **Useful References:**

* 🌐 [Bitnami Helm Charts](https://bitnami.com/stacks/helm)
* 🔎 [ArtifactHub - Helm Chart Search](https://artifacthub.io/)

```bash
# List current Helm repositories
helm repo list

# Add Bitnami Helm repository
helm repo add mybitnami https://charts.bitnami.com/bitnami

# Confirm it's added
helm repo list

# Search for charts in the repository
helm search repo nginx
helm search repo apache
helm search repo wildfly
```

---

## 🚀 Step 03: Install a Helm Chart

🛠️ We’ll install the `nginx` chart from Bitnami’s Helm repository.

```bash
# Update the repo to get latest chart versions
helm repo update

# Install the chart
helm install mynginx mybitnami/nginx
```

---

## 📋 Step 04: List Helm Releases

View current deployments and output in multiple formats:

```bash
# List releases (default output)
helm list
helm ls

# List releases in YAML format
helm list --output=yaml

# List releases in JSON format
helm list --output=json

# List releases within a namespace
helm list --namespace=default
helm list -n default
```

---

## 🧾 Step 05: Access Kubernetes Resources

Inspect services and pods, then port-forward the service for local access.

```bash
# List deployed pods
kubectl get pods

# List services
kubectl get svc

# Port-forward nginx service to your localhost
kubectl port-forward service/mynginx 8088:80
```

🌐 **Access the NGINX application locally:**

* Browser:
  [http://localhost:8088](http://localhost:8088)
  [http://127.0.0.1:8088](http://127.0.0.1:8088)

* Curl:

```bash
curl http://localhost:8088
curl http://127.0.0.1:8088
```

---

## 🧹 Step 06: Uninstall Helm Release

Clean up the release:

```bash
# List current Helm releases
helm list

# Uninstall the nginx release
helm uninstall mynginx
```

---

## ✅ Summary

| Step     | Command                | Description                     |
| -------- | ---------------------- | ------------------------------- |
| Add Repo | `helm repo add`        | Add a new Helm chart repository |
| Search   | `helm search repo`     | Find charts in added repos      |
| Install  | `helm install`         | Deploy an application           |
| List     | `helm list`            | View deployed Helm releases     |
| Access   | `kubectl port-forward` | Access app via local browser    |
| Remove   | `helm uninstall`       | Uninstall a Helm release        |

---