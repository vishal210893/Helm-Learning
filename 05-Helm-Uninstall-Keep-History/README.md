# 🧹 Helm Uninstall – *Keep History Best Practice*

> Learn how to **uninstall a Helm release without losing its revision history**, allowing future rollbacks and auditability.

---

## 📘 Step 01: Introduction

This guide focuses on:

* Safely uninstalling Helm releases
* Preserving release history for rollback or audit purposes
* Demonstrating the **difference between uninstalling with and without `--keep-history`**

> ⚠️ **Note**: This demo assumes you’ve already deployed a Helm release like `myapp101`.

---

## 🗂️ Step 02: Uninstall Helm Release with `--keep-history`

```bash
# 🔍 View Helm releases
helm list
helm list --superseded
helm list --deployed

# 📜 View revision history of a release
helm history myapp101

# 🧹 Uninstall but keep the history
helm uninstall myapp101 --keep-history

# 🔎 List uninstalled releases
helm list --uninstalled
```

> ✅ **Expected**:

* `myapp101` should be listed under uninstalled releases.
* `helm status` should still work.

```bash
# 🧾 View status of the uninstalled release
helm status myapp101
```

📌 **Observation**:

* Works **only when** `--keep-history` is used
* Output shows `Status: uninstalled`, but retains all previous data

---

## 🔁 Step 03: Rollback Uninstalled Release

```bash
# 📜 Review revision history
helm history myapp101

# ⏪ Rollback to a previous revision
helm rollback myapp101 3
```

> ✅ **Expected**: Helm redeploys the release at **revision 3** using retained metadata.

```bash
# 🔍 Confirm the release and K8s resources
helm list
kubectl get pods
kubectl get svc

# 🔎 View full resource info
helm status myapp101 --show-resources
```

🌐 **Access the Application**:

```
http://localhost:31232
```

---

## ❌ Step 04: Uninstall Without `--keep-history`

```bash
# 🔍 View releases
helm list

# 🚫 Uninstall permanently (no history retained)
helm uninstall myapp101
```

```bash
# 🔍 Try to list uninstalled releases
helm list --uninstalled
```

📌 **Observation**:

* Release is **not listed**
* `helm status` will fail

```bash
# 🚫 This will throw an error now
helm status myapp101

# 🚫 History is also gone
helm history myapp101
```

---

## 🔒 Step 05: Attempt Rollback After Permanent Uninstall

```bash
# 🚫 Try to rollback a fully removed release
helm rollback myapp101 1
```

📌 **Expected Error**:

```
Error: release: not found
```

> ❗ Helm has no memory of this release anymore, as all metadata has been removed.

---

## ✅ Step 06: Best Practice — Use `--keep-history`

### ✅ Why You Should Use `--keep-history`:

* 🔁 Enables **rollback of previously uninstalled** releases
* 📜 Preserves **audit trail and history**
* 🧪 Useful for testing and reverting deployments
* 🧘 Safer during experimentation or CI/CD cycles

```bash
helm uninstall myapp101 --keep-history
```

---

## 📌 Summary Table

| Action                        | Command Example                          | Outcome                               |
| ----------------------------- | ---------------------------------------- | ------------------------------------- |
| Uninstall (keep history)      | `helm uninstall myapp101 --keep-history` | Status: uninstalled, rollbackable     |
| View release status (kept)    | `helm status myapp101`                   | Shows release with uninstalled status |
| Rollback to revision (kept)   | `helm rollback myapp101 3`               | Restores release to revision 3        |
| Uninstall (no flags)          | `helm uninstall myapp101`                | Permanently deleted                   |
| Rollback after full uninstall | `helm rollback myapp101 1`               | ❌ Throws "release: not found"         |

---