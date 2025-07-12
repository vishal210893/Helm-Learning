# 💣 Helm Install with `--atomic` Flag

> Ensure *clean rollbacks* on failure! Learn how to use Helm's `--atomic` flag to avoid leaving failed releases behind.

---

## 📘 Step 01: Introduction

The `--atomic` flag is a powerful option when using `helm install`:

* Automatically **rolls back** a failed installation
* Prevents cluttering your cluster with failed releases
* Activates the `--wait` flag by default to ensure all components are healthy before considering a release successful

✅ **Best suited for**:

* CI/CD pipelines
* Production environments
* Repeatable automated deployments

---

## 🚀 Step 02: Install Helm Chart (Release: `dev101`)

```bash
# Install chart normally (without atomic)
helm install dev101 stacksimplify/mychart1

# List installed releases
helm list

# Check status and deployed resources
helm status dev101 --show-resources
```

🌐 **Access the application**:

```
http://localhost:31231
```

---

## 💥 Step 03: Simulate Installation Failure (Release: `qa101`)

```bash
# Try installing a second release using the same nodePort (will fail)
helm install qa101 stacksimplify/mychart1
```

📌 **Expected Error**:

```
INSTALLATION FAILED: Service "qa101-mychart1" is invalid:
spec.ports[0].nodePort: Invalid value: 31231: provided port is already allocated
```

```bash
# List releases — you'll see 'qa101' with FAILED status
helm list
```

🧹 **Cleanup the failed release manually**:

```bash
helm uninstall qa101
```

---

## 🛡️ Step 04: Use `--atomic` to Auto-Rollback on Failure

```bash
# Install with atomic rollback
helm install qa101 stacksimplify/mychart1 --atomic
```

📌 **Expected Behavior**:

* Installation fails (due to the same port conflict)
* Helm **automatically deletes** the failed release
* `helm list` will **not** show `qa101` at all

⚠️ **Error still appears** for visibility:

```
INSTALLATION FAILED: Service "qa101-mychart1" is invalid:
spec.ports[0].nodePort: Invalid value: 31231: provided port is already allocated
```

⏱️ The following flags are automatically triggered by `--atomic`:

* `--wait`: Waits for pods and services to become ready
* `--timeout`: Defaults to 5m unless otherwise specified

---

## 🧹 Step 05: Uninstall the `dev101` Release

```bash
# Clean up original release
helm uninstall dev101

# Confirm removal
helm list
```

---

## ✅ Summary

| Feature             | Command Example                                      | Behavior / Outcome                      |
| ------------------- | ---------------------------------------------------- | --------------------------------------- |
| Normal Install      | `helm install dev101 stacksimplify/mychart1`         | Installs the chart                      |
| Failure (No Atomic) | `helm install qa101 stacksimplify/mychart1`          | Leaves release in FAILED state          |
| Atomic Install      | `helm install qa101 stacksimplify/mychart1 --atomic` | Rolls back failed install automatically |
| Cleanup             | `helm uninstall <release-name>`                      | Removes installed release               |

---

## 💡 Best Practice Tip

In automated environments (CI/CD), always use:

```bash
helm install <release> <chart> --atomic
```

to:

* Avoid leftover failed states
* Ensure cluster consistency
* Safeguard against partial deployments

---
