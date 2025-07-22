# Helm Hooks

## Step-01: Introduction

* Helm hooks allow chart developers to intervene at certain points in a release lifecycle. These are useful for tasks like initializing databases, running preflight checks, cleanup jobs, etc.

* Hooks are declared using the `helm.sh/hook` annotation.

---

## Step-02: Create a Simple Chart Using Starter Chart

> **Note:** You can skip this step if you already have the `hooksdemo1` chart prepared.

```bash
# Generate a new chart using your custom starter chart
helm create hooksdemo1 --starter=mystarterchart
```

---

## Step-03: Define a Pre-install Hook

**File:** `templates/preinstall-hookpod.yaml`

```yaml
apiVersion: v1
kind: Pod
metadata: 
  name: myhook-preinstall
  annotations:
    "helm.sh/hook": "pre-install"
spec:
  restartPolicy: Never
  containers:
    - name: myhook-preinstall-container
      image: busybox
      imagePullPolicy: IfNotPresent
      command: ['sh', '-c', 'echo Pre-install hook Pod is running && sleep 15']
```

---

## Step-04: Define a Pre-upgrade Hook

**File:** `templates/preupgrade-hookpod.yaml`

```yaml
apiVersion: v1
kind: Pod
metadata: 
  name: myhook-preupgrade
  annotations:
    "helm.sh/hook": "pre-upgrade"
spec:
  restartPolicy: Never
  containers:
    - name: myhook-preupgrade-container
      image: busybox
      imagePullPolicy: IfNotPresent
      command: ['sh', '-c', 'echo preupgrade hook Pod is running && sleep 15']
```

---

## Step-05: Define a Post-delete Hook

**File:** `templates/postdelete-hookpod.yaml`

```yaml
apiVersion: v1
kind: Pod
metadata: 
  name: myhook-postdelete
  annotations:
    "helm.sh/hook": "post-delete"
spec:
  restartPolicy: Never
  containers:
    - name: myhook-postdelete-container
      image: busybox
      imagePullPolicy: IfNotPresent
      command: ['sh', '-c', 'echo post-delete hook Pod is running && sleep 15']
```

---

## Step-06: Test Pre-install Hook

```bash
cd hooksdemo1

# Install release
helm install myapp101 . --atomic

# Check all pods including hook pod
kubectl get pods

# Describe hook pod
kubectl describe pod myhook-preinstall | grep -E 'Anno|Started:|Finished:'

# Verify main application pod
kubectl get pods
kubectl describe pod $(kubectl get pods -l app.kubernetes.io/name=hooksdemo1 -o name) | grep -E 'Anno|Started:|Finished:'

# Access app
kubectl get svc
# Access app at: http://localhost:31239
```

**Observation:**

* The `myhook-preinstall` pod runs first and completes before the release is fully deployed.
* Application should reflect v1 version.

---

## Step-07: Understand Hook Lifecycle

* If both `pre-install` and `post-install` hooks are defined, Helm runs the hooks before and after installing templates respectively.

* Refer to Helm docs on [hooks and the release lifecycle](https://helm.sh/docs/topics/charts_hooks/#hooks-and-the-release-lifecycle) for timing details and execution order.

---

## Step-08: Test Pre-upgrade Hook

```bash
# Perform upgrade with a new image tag
helm upgrade myapp101 . --set image.tag=0.2.0

# Observe hook pod
kubectl get pods
kubectl describe pod myhook-preupgrade | grep -E 'Anno|Started:|Finished:'

# Check app pod
kubectl describe pod $(kubectl get pods -l app.kubernetes.io/name=hooksdemo1 -o name) | grep -E 'Anno|Started:|Finished:'

# Access app
kubectl get svc
# Access app at: http://localhost:31239
```

**Observation:**

* A new pod `myhook-preupgrade` is triggered during upgrade and completes before app update.
* Application should reflect v2 version.

---

## Step-09: Test Post-delete Hook

```bash
# Uninstall release
helm uninstall myapp101

# Verify pods
kubectl get pods
```

**Observation:**

* You should still see `myhook-postdelete` pod in completed status.
* All 3 hook pods remain present even after uninstall.

---

## Step-10: Hooks are Not Part of Release Resources

* Hook-generated resources are **not tracked** as part of the release.
* Helm will wait for them to complete (if applicable), but it will **not delete** them on `helm uninstall`.

> ⚠️ This behavior means hook pods **must be manually cleaned up** if needed after their execution.

---
