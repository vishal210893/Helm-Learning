# 🚀 Helm Upgrade with `--set` Option

> Upgrade Helm releases dynamically using the `--set` flag to override values like Docker image tags.

---

## 📘 Introduction

This guide demonstrates how to:

* Install and upgrade a Helm release
* Use `--set "image.tag=<DOCKER-IMAGE-TAG>"` to upgrade images
* Explore useful Helm commands

💡 Helm Commands covered:

* `helm repo`
* `helm search repo`
* `helm install`
* `helm upgrade`
* `helm history`
* `helm status`

---

## 📦 Custom Helm Repository

### 🔍 Review Our Custom Helm Repo

* 📦 [StackSimplify Helm Repo](https://stacksimplify.github.io/helm-charts/)
* 💻 [GitHub - stacksimplify/helm-charts](https://github.com/stacksimplify/helm-charts)
* 🔎 Search on [artifacthub.io](https://artifacthub.io) → `stacksimplify`
* 📜 [mychart1 - ArtifactHub](https://artifacthub.io/packages/helm/stacksimplify/mychart1)

---

### ➕ Add the Custom Helm Repo

```bash
# List existing Helm repositories
helm repo list

# Add the custom repository
helm repo add stacksimplify https://stacksimplify.github.io/helm-charts/

# Refresh Helm repo cache
helm repo update

# Search chart in repo
helm search repo mychart1
```

---

## 🚀 Install Helm Chart

```bash
# Install chart from custom repo
helm install myapp1 stacksimplify/mychart1
```

---

## 🔍 Post-Install: List & Access Resources

```bash
# List Helm releases
helm list

# Get running pods
kubectl get pods

# View services
kubectl get svc

# Access the app (via NodePort)
http://localhost:<NODE-PORT>
# Example:
http://localhost:31231
```

---

## 🔁 Helm Upgrade Using --set

* Docker Image: [`kubenginx`](https://github.com/users/stacksimplify/packages/container/package/kubenginx)
* Available Tags: `1.0.0`, `2.0.0`, `3.0.0`, `4.0.0`

```bash
# Upgrade image version via --set
helm upgrade myapp1 stacksimplify/mychart1 --set "image.tag=2.0.0"
```

---

## 🧾 Validate Upgrade Resources

```bash
# List releases - revision should increment
helm list

# List superseded or deployed releases
helm list --superseded
helm list --deployed

# Inspect pod and verify image version
kubectl get pods
kubectl describe pod <POD-NAME>

# App access (check updated version)
http://localhost:31231
```

---

## 🔁 Practice: Additional Helm Upgrades

```bash
# Upgrade to version 3.0.0
helm upgrade myapp1 stacksimplify/mychart1 --set "image.tag=3.0.0"

# Upgrade to version 4.0.0
helm upgrade myapp1 stacksimplify/mychart1 --set "image.tag=4.0.0"
```

---

## 🕓 Helm History

```bash
# View historical revisions
helm history myapp1
```

---

## 📊 Helm Status

```bash
# View current status of release
helm status myapp1

# Show description
helm status myapp1 --show-desc

# Show resources managed by the release
helm status myapp1 --show-resources

# View status for a specific revision
helm status myapp1 --revision 2
```

---

## 🧹 Cleanup: Uninstall Release

```bash
# Remove Helm release
helm uninstall myapp1
```

---

## ⚙️ Optional: Bash Script for Upgrade

Create a file called `helm-upgrade.sh`:

```bash
#!/bin/bash

# Usage: ./helm-upgrade.sh <release-name> <image-tag>

RELEASE_NAME=$1
IMAGE_TAG=$2
CHART_NAME="stacksimplify/mychart1"

if [[ -z "$RELEASE_NAME" || -z "$IMAGE_TAG" ]]; then
  echo "Usage: $0 <release-name> <image-tag>"
  exit 1
fi

echo "Upgrading $RELEASE_NAME to image tag $IMAGE_TAG..."
helm upgrade "$RELEASE_NAME" "$CHART_NAME" --set "image.tag=$IMAGE_TAG"

echo "Upgrade complete. Checking pod status..."
kubectl get pods -l app.kubernetes.io/instance=$RELEASE_NAME
```

> Make it executable:

```bash
chmod +x helm-upgrade.sh
```

> Run it:

```bash
./helm-upgrade.sh myapp1 3.0.0
```

---

## ✅ Summary

| Step     | Command                        | Description                        |
| -------- | ------------------------------ | ---------------------------------- |
| Add Repo | `helm repo add`                | Add custom repo to Helm            |
| Install  | `helm install`                 | Deploy initial release             |
| Upgrade  | `helm upgrade --set`           | Upgrade release with new image tag |
| Inspect  | `helm list / history / status` | Monitor status and changes         |
| Cleanup  | `helm uninstall`               | Remove release                     |

---