# 📦 Helm with Kubernetes Namespaces

> Learn how to deploy, upgrade, and uninstall Helm releases to specific Kubernetes namespaces using `--namespace` and `--create-namespace` flags.

---

## 📘 Step 01: Introduction

Helm deploys Kubernetes resources **within a namespace context**.

### 🔑 Key Points:

* 📌 **Default namespace** is used if none is specified.
* 📂 You can deploy to any namespace using:

    * `--namespace <name>` or `-n <name>`
* 🏗️ You can also **create the namespace** automatically during install using:

    * `--create-namespace`

> ✅ This is especially helpful in multitenant clusters or isolated staging environments.

---

## 🚀 Step 02: Install Helm Release into a New Namespace (`dev`)

```bash
# 🔍 List current namespaces
kubectl get ns

# 📥 Install Helm chart and auto-create namespace 'dev'
helm install dev101 stacksimplify/mychart2 \
  --version "0.1.0" \
  --namespace dev \
  --create-namespace
```

```bash
# 🔍 Confirm 'dev' namespace was created
kubectl get ns
```

```bash
# 📋 Check Helm releases
helm list                # Default namespace: should show nothing
helm list -n dev         # Our release will appear here
helm list --namespace dev
```

```bash
# 🔎 Get status and resources deployed
helm status dev101 --show-resources -n dev
```

```bash
# 🧾 List Kubernetes resources in dev namespace
kubectl get pods -n dev
kubectl get svc -n dev
kubectl get deploy -n dev
```

🌐 **Access the application**:

```
http://localhost:31232
```

---

## 🔁 Step 03: Upgrade Helm Release in `dev` Namespace

```bash
# ⬆️ Upgrade to a new chart version in 'dev'
helm upgrade dev101 stacksimplify/mychart2 \
  --version "0.2.0" \
  --namespace dev
```

```bash
# 📋 Verify release and resource status
helm list -n dev
helm status dev101 --show-resources -n dev
```

🌐 **Access the upgraded application**:

```
http://localhost:31232
```

---

## 🧹 Step 04: Uninstall Helm Release from `dev` Namespace

```bash
# ❌ Uninstall Helm release from dev
helm uninstall dev101 --namespace dev
```

```bash
# 📋 Confirm it's gone
helm list -n dev
```

```bash
# 🔍 Check if the namespace still exists
kubectl get ns
```

📌 **Observation**:

* Helm does **not delete the namespace**.
* If it's no longer needed, manually delete it:

```bash
# 🗑️ Delete the 'dev' namespace
kubectl delete ns dev

# 🔄 Verify deletion
kubectl get ns
```

---

## ✅ Summary

| Action                           | Command Example                                         | Description                                      |
| -------------------------------- | ------------------------------------------------------- | ------------------------------------------------ |
| Install to namespace             | `helm install dev101 <chart> -n dev --create-namespace` | Deploys to `dev` and creates namespace if needed |
| List releases in namespace       | `helm list -n dev`                                      | Shows releases only in specified namespace       |
| Get status in namespace          | `helm status dev101 -n dev --show-resources`            | Shows resource state in the namespace            |
| Upgrade in namespace             | `helm upgrade dev101 <chart> --version <v> -n dev`      | Upgrades release in `dev`                        |
| Uninstall release from namespace | `helm uninstall dev101 -n dev`                          | Removes release but **not** the namespace        |
| Delete namespace (optional)      | `kubectl delete ns dev`                                 | Clean up the namespace manually                  |

---

## 💡 Best Practice Tip

When designing **CI/CD workflows**, always:

* Use **namespaced releases** for isolation
* Prefer `--create-namespace` for idempotency
* Clean up with `kubectl delete ns <name>` only when sure nothing else lives in that namespace

---