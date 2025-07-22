# Helm Hook Weights

## Step-01: Introduction

* Helm supports ordering of hooks using **hook weights**, allowing precise control over the sequence of hook execution.

* Hook weights must be provided as **string values**, even if the value is a number:

  ```yaml
  annotations:
    "helm.sh/hook-weight": "5"
  ```

* The execution order for hooks of the **same lifecycle event** (e.g., `pre-install`) is determined by sorting these weights in **ascending** order.

---

## Step-02: Review Hook Pod Template Annotations

To demonstrate hook weights, three pre-install hook pods are defined with different weights:

### preinstall-hookpod1.yaml

```yaml
annotations:
  "helm.sh/hook": "pre-install"
  "helm.sh/hook-delete-policy": before-hook-creation
  "helm.sh/hook-weight": "-2"
```

### preinstall-hookpod2.yaml

```yaml
annotations:
  "helm.sh/hook": "pre-install"
  "helm.sh/hook-delete-policy": before-hook-creation
  "helm.sh/hook-weight": "5"
```

### preinstall-hookpod3.yaml

```yaml
annotations:
  "helm.sh/hook": "pre-install"
  "helm.sh/hook-delete-policy": before-hook-creation
  "helm.sh/hook-weight": "6"
```

> Note: All three hooks are for the same lifecycle event (`pre-install`) and are of the same Kubernetes resource type (`Pod`).

---

## Step-03: Install Helm Release

```t
# Change Directory (In Helm Chart Folder)
cd hooksdemo1

# Install the Helm Release
helm install myapp101 .
```

**Observation:**

* All three hook pods will be created and completed in sequence based on their weights.
* Execution order:

    1. `myhook-preinstall1` (hook-weight: -2)
    2. `myhook-preinstall2` (hook-weight: 5)
    3. `myhook-preinstall3` (hook-weight: 6)

```t
# List Hook Pods
kubectl get pods

# Confirm Execution Order by Checking Timestamps
kubectl describe pod myhook-preinstall1 | grep -E 'Anno|Started:|Finished:'
kubectl describe pod myhook-preinstall2 | grep -E 'Anno|Started:|Finished:'
kubectl describe pod myhook-preinstall3 | grep -E 'Anno|Started:|Finished:'
```

This helps confirm that **hooks with lower weights run before those with higher weights**.

---

## Step-04: Uninstall Helm Release and Clean-Up

```t
# Uninstall the Helm Release
helm uninstall myapp101

# Manually Delete Residual Hook Pods (if still present)
kubectl get pods
kubectl delete pod myhook-preinstall1
kubectl delete pod myhook-preinstall2
kubectl delete pod myhook-preinstall3
```

By properly configuring **`hook-weight`**, you can orchestrate complex pre/post operations in your Helm deployments while maintaining a predictable and controlled execution flow.
