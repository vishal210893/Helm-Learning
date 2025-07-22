# Helm Hooks Delete Policy

## Step-01: Introduction

* Implement Helm Hooks deletion policy to automatically clean up hook-created resources based on lifecycle conditions.

## Step-02: List Kubernetes Pods

* **Important Note:** This step continues from the previous `hooksdemo1` demo.

```t
# List Kubernetes Pods
kubectl get pods
```

**Observation:**

1. Previously created hook pods (like `myhook-preinstall`, `myhook-preupgrade`, `myhook-postdelete`) are in `Completed` status and still present.
2. Options to remove them:

    * **Option 1:** Manually delete the pods.
    * **Option 2:** Use **Helm Hook Deletion Policies** to automate cleanup.

---

## Step-03: What are Helm Hook Deletion Policies?

Helm provides a way to specify when a hook resource should be deleted using the `helm.sh/hook-delete-policy` annotation:

```yaml
annotations:
  "helm.sh/hook-delete-policy": before-hook-creation,hook-succeeded,hook-failed
```

Supported policies:

| Policy                 | Description                                                             |
| ---------------------- | ----------------------------------------------------------------------- |
| `before-hook-creation` | Deletes the previous hook resource before creating a new one (default). |
| `hook-succeeded`       | Deletes the resource if the hook runs successfully.                     |
| `hook-failed`          | Deletes the resource if the hook fails.                                 |

---

## Step-04: Deploy New Helm Release

```t
# Change to Helm Chart Directory
cd hooksdemo1

# Note existing hook pod ages
kubectl get pods

# Install Helm Release
helm install myapp101 .
```

**Observation:**

1. Existing `myhook-preinstall` pod is deleted and re-created automatically.
2. This is due to the default `before-hook-creation` policy, even though it's not explicitly defined.

---

## Step-05: Uninstall Helm Release and Clean Up

```t
# Uninstall Helm Release
helm uninstall myapp101

# Delete remaining hook pods manually (if present)
kubectl delete pod myhook-preinstall
kubectl delete pod myhook-preupgrade
kubectl delete pod myhook-postdelete
```

---

## Step-06: Update HookPod YAML Files with Hook Deletion Policy

Update the following files with `helm.sh/hook-delete-policy: before-hook-creation,hook-succeeded`:

* `templates/preinstall-hookpod.yaml`
* `templates/preupgrade-hookpod.yaml`
* `templates/postdelete-hookpod.yaml`

```yaml
annotations:
  "helm.sh/hook": "pre-install"
  "helm.sh/hook-delete-policy": before-hook-creation,hook-succeeded
```

---

## Step-07: Install Helm Release and Test Hook Deletion Policy

```t
# Install Helm Release
helm install myapp101 .
```

**Observation:**

* `myhook-preinstall` pod is created, completed, and automatically deleted due to `hook-succeeded` policy.

---

## Step-08: Upgrade Helm Release and Test Hook Deletion Policy

```t
# Upgrade Helm Release
helm upgrade myapp101 . --set image.tag=0.2.0
```

**Observation:**

* `myhook-preupgrade` pod is created, completed, and then deleted due to `hook-succeeded` policy.

---

## Step-09: Uninstall Helm Release and Test Hook Deletion Policy

```t
# Uninstall Helm Release
helm uninstall myapp101
```

**Observation:**

* `myhook-postdelete` pod is created and cleaned up after successful execution.

---

## Step-10: Downside of Using `hook-failed`

* While `hook-failed` is useful in production for auto-cleanup of failed resources, it can hinder debugging during chart development.
* Without the resource present, it becomes harder to inspect logs or describe the failure.
* **Recommendation:** Avoid `hook-failed` during development to retain hook artifacts for troubleshooting.
