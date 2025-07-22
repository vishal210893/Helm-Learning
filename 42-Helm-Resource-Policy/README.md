# Helm Resource Policy Demo

## Step-01: Introduction

Helm provides a way to prevent certain Kubernetes resources from being deleted during operations such as `helm uninstall`, `helm upgrade`, or `helm rollback`. This is achieved using the annotation:

```yaml
"helm.sh/resource-policy": keep
```

### Key Points:

* Adding this annotation to a resource (like a Deployment) tells Helm to **retain** the resource even if the chart is deleted.
* Helm will **no longer manage** the resource after it’s marked to keep — it becomes an **orphan**.
* This can **cause conflicts** with future `helm install --replace` operations, especially if the retained resource still exists with the same name.

---

## Step-02: Helm Resource Policy Annotation

To apply the resource retention behavior, use the following annotation in the metadata block of a Helm-managed resource:

```yaml
metadata:
  annotations:
    "helm.sh/resource-policy": keep
```

This is typically added in the `deployment.yaml` or other template files.

---

## Step-03: Create a Helm Chart and Apply Resource Policy

```t
# Create a new Helm chart
helm create respolicytest
```

Update the `respolicytest/templates/deployment.yaml` file:

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: {{ include "respolicytest.fullname" . }}
  labels:
    {{- include "respolicytest.labels" . | nindent 4 }}
  annotations:
    "helm.sh/resource-policy": keep
spec:
  ...
```

**Note:** The rest of the deployment spec remains unchanged.

---

## Step-04: Install, Uninstall, and Verify Behavior

```t
# Navigate to the chart directory
cd respolicytest

# Install the chart
helm install myapp1 .

# Verify deployed resources
kubectl get deploy
kubectl get pods
kubectl get svc
```

### Observation:

* Deployment and its associated pods and services should be created as usual.

```t
# Uninstall the Helm release
helm uninstall myapp1

# Check the state of resources
kubectl get deploy
kubectl get pods
kubectl get svc
```

### Post-Uninstall Observation:

* The **Deployment** should still exist (was not removed).
* The **Pods** related to that Deployment will remain in a running state.
* Helm will not track this resource anymore — it’s now orphaned.

---

## Step-05: Clean Up Orphaned Resources

To manually clean up the retained resources:

```t
kubectl delete deploy myapp1-respolicytest
```

This step is necessary to ensure no lingering orphaned resources interfere with future Helm installations.

---
