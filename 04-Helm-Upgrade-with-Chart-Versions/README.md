# 🚀 Helm Upgrade Using Specific Chart Versions

> Learn how to install, upgrade, and roll back Helm releases using specific **chart versions**, along with exploring Helm history and release status.

---

## 📘 Step 01: Introduction

This guide covers:

* Installing specific versions of a Helm chart
* Upgrading to a newer chart version
* Viewing release history and status
* Performing rollbacks to earlier versions

### 🔧 Helm Commands Used:

* `helm install`
* `helm search repo`
* `helm status`
* `helm upgrade`
* `helm rollback`
* `helm history`

---

## 📦 Step 02: Search for Chart Versions (`mychart2`)

🔗 References:

* 📂 [GitHub – stacksimplify/mychart2](https://github.com/stacksimplify/helm-charts/tree/main/mychart2)
* 📦 [ArtifactHub – mychart2](https://artifacthub.io/packages/helm/stacksimplify/mychart2/)

`mychart2` Chart Version → App Version Mapping:

| Chart Version | App Version |
| ------------- | ----------- |
| 0.1.0         | 1.0.0       |
| 0.2.0         | 2.0.0       |
| 0.3.0         | 3.0.0       |
| 0.4.0         | 4.0.0       |

```bash
# 🔍 View the latest available version
helm search repo mychart2

# 🔍 View all available versions of the chart
helm search repo mychart2 --versions

# 🔍 Filter for a specific chart version
helm search repo mychart2 --version "0.2.0"
```

> 💡 **Observation**: These commands help verify the available versions before installation or upgrade.

---

## 🚀 Step 03: Install a Specific Chart Version

```bash
# 📥 Install a specific version of the chart
helm install myapp101 stacksimplify/mychart2 --version "0.1.0"

# 📋 View installed Helm releases
helm list

# 🔎 Inspect deployed resources
helm status myapp101 --show-resources
```

🔗 **Access the Application:**

```
http://localhost:31232
```

```bash
# 🔧 View logs of the deployed pod
kubectl get pods
kubectl logs -f <POD-NAME>
```

---

## 🔁 Step 04: Upgrade Helm Release with a Specific Chart Version

```bash
# ⬆️ Upgrade to a newer chart version
helm upgrade myapp101 stacksimplify/mychart2 --version "0.2.0"

# 🔍 Check upgrade result
helm list
helm status myapp101 --show-resources
helm history myapp101
```

🔗 **Verify Updated App:**

```
http://localhost:31232
```

> ✅ **Expectation**: App Version should now reflect `2.0.0`

---

## ⏫ Step 05: Upgrade to Latest Chart Version (Without `--version`)

```bash
# ⬆️ Upgrade to latest available chart version
helm upgrade myapp101 stacksimplify/mychart2

# 🔍 View release details and history
helm status myapp101 --show-resources
helm history myapp101
```

> ⚠️ **Observation**: Upgrades to **latest chart version** → Chart `0.4.0`, App `4.0.0`.

🔗 **Test the Application:**

```
http://localhost:31232
```

---

## 🔙 Step 06: Roll Back to Previous Revision

Rolls back to the release's immediate previous version.

```bash
# ⏪ Roll back to previous version
helm rollback myapp101

# 🔍 Review state
helm status myapp101 --show-resources
helm history myapp101
```

> ✅ **Expectation**: Application reverts to **Chart Version 0.2.0**, App Version `2.0.0`

---

## 🎯 Step 07: Roll Back to Specific Revision

```bash
# ⏮️ Rollback to revision 1 (initial install)
helm rollback myapp101 1

# 🔍 Review release info
helm list
helm status myapp101 --show-resources
helm history myapp101
```

> ✅ **Expectation**: Application should now show **Chart Version 0.1.0**, App Version `1.0.0`

---

## ✅ Summary

| Step                | Command Example                             | Description                      |
| ------------------- | ------------------------------------------- | -------------------------------- |
| Search              | `helm search repo mychart2 --versions`      | View available chart versions    |
| Install             | `helm install myapp101 ... --version 0.1.0` | Install specific chart version   |
| Upgrade             | `helm upgrade myapp101 ... --version 0.2.0` | Upgrade to a specific version    |
| Upgrade (Latest)    | `helm upgrade myapp101 ...`                 | Upgrade to the latest version    |
| History             | `helm history myapp101`                     | View revision history            |
| Rollback            | `helm rollback myapp101`                    | Rollback to previous version     |
| Rollback (Specific) | `helm rollback myapp101 1`                  | Rollback to a specified revision |

---