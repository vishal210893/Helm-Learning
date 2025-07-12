# 🚀 Docker Desktop + Kubernetes + Helm CLI Setup Guide

## 📌 Step 01: Overview

This guide will help you:

* Install Docker Desktop (Mac/Windows)
* Enable Kubernetes in Docker Desktop
* Verify with sample application deployment
* Install Helm CLI using your OS package manager
* Setup `kubectl` context and configuration

---

## 🐳 Step 02: Docker Desktop — Pricing, Signup, and Download

* [Docker Desktop Pricing](https://www.docker.com/pricing/)
* [Sign Up on Docker Hub](https://hub.docker.com/)
* [Download Docker Desktop](https://www.docker.com/products/docker-desktop/)

> ✅ **Tip:** Docker Desktop requires a Docker Hub login for license validation after installation. Login with your Docker ID after setup.

---

## 💻 Step 03: Install Docker Desktop

### 🖥️ macOS

```bash
# 1. Download Docker Desktop for macOS (Intel or Apple Silicon)
# 2. Drag the downloaded .dmg to the Applications folder
# 3. Open Docker Desktop and Sign In with your Docker Hub credentials
```

### 🪟 Windows

```bash
# 1. Download Docker Desktop Installer
# 2. Run: Docker Desktop Installer.exe
# 3. Sign In to Docker Hub

# Optional: Add Docker binaries to system PATH
C:\Program Files\Docker\Docker\Resources\bin
```

> ⚠️ **Note for Windows Home:** Enable WSL 2 backend and install Linux kernel if required.

---

## ☸️ Step 04: Enable Kubernetes in Docker Desktop

* Navigate to `Settings` → `Kubernetes` → Check **Enable Kubernetes**
* Click **Apply & Restart**
* Wait for 5–10 minutes for the Kubernetes cluster to be provisioned

🔗 [Official Docs](https://docs.docker.com/desktop/kubernetes/)

---

## 🔧 Step 05: Configure `kubectl` for Docker Desktop K8s Cluster

```bash
# Check if kubectl is available
which kubectl

# Verify kubectl version
kubectl version --short
kubectl version --client --output=yaml

# View all contexts
kubectl config get-contexts

# Set current context (if needed)
kubectl config use-context docker-desktop

# Validate cluster status
kubectl get nodes
```

> 🧠 **Pro Tip:** Save your working kubeconfig with `kubectl config view --flatten > ~/.kube/config.backup`

---

## ✅ Step 06: Verify Cluster with Sample Application

### 📁 Manifests Structure

```plaintext
kube-manifests/
├── deployment.yaml
└── service.yaml
```

### 🚀 Deploy & Access App

```bash
# Apply manifests
kubectl apply -f kube-manifests/

# Get Deployment/Pods/Services
kubectl get deploy
kubectl get pods
kubectl get svc

# Access via NodePort
http://localhost:31300
# or
http://127.0.0.1:31300
```

### 🧹 Cleanup

```bash
kubectl delete -f kube-manifests/
kubectl get pods,svc,deploy
```

> 🔎 **Troubleshooting Tip:** Use `kubectl describe pod <name>` or `kubectl logs <pod>` for debugging.

---

## 📦 Step 07: Install Helm CLI (Cross-Platform)

🔗 [Official Helm Install Docs](https://helm.sh/docs/intro/install/)

### 🖥️ macOS

```bash
brew install helm
```

### 🪟 Windows (via Chocolatey or Scoop)

```bash
choco install kubernetes-helm
# or
scoop install helm
```

### 📦 Download Binary (All OS)

* [Helm GitHub Releases](https://github.com/helm/helm/releases)

> Extract manually and add to PATH.

### 🧪 Verify Helm

```bash
helm version
helm env
```

> 🧠 **Bonus:** Set `HELM_DEBUG=true` during troubleshooting to print verbose output.

---

## 🪟 Step 08: Windows - Manual Helm CLI Install (If No Package Manager)

```bash
# Download zip
https://github.com/helm/helm/releases

# Extract
helm-v3.12.3-windows-amd64.zip → C:\helm\windows-amd64

# Add to PATH
C:\helm\windows-amd64
```

---

## ⛑️ Optional: Install/Update `kubectl`

🔗 [kubectl Install Instructions](https://kubernetes.io/docs/tasks/tools/)

### 🖥️ macOS

```bash
# For Intel
curl -LO "https://dl.k8s.io/release/v1.27.2/bin/darwin/amd64/kubectl"

# For Apple Silicon
curl -LO "https://dl.k8s.io/release/v1.27.2/bin/darwin/arm64/kubectl"

chmod +x ./kubectl
sudo mv ./kubectl /usr/local/bin/kubectl
```

### 🪟 Windows

Use `choco install kubernetes-cli` or manual install from:

* [kubectl Windows Downloads](https://kubernetes.io/docs/tasks/tools/install-kubectl-windows/)

---

## 🔐 Developer Extras & Best Practices

| Feature/Tip                 | Command / Link                                   |
| --------------------------- | ------------------------------------------------ |
| Switch context              | `kubectl config use-context`                     |
| View current context        | `kubectl config current-context`                 |
| Helm repositories           | `helm repo list`, `helm repo add`                |
| Helm charts discovery       | [https://artifacthub.io](https://artifacthub.io) |
| Troubleshoot pods           | `kubectl describe pod <name>`                    |
| Helm upgrade with rollback  | `helm upgrade --install --atomic`                |
| Store kubeconfig separately | `KUBECONFIG=./kubeconfig.yaml kubectl get pods`  |
| Monitor resources           | `kubectl top pod` (via metrics-server)           |

---