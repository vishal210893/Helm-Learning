# 🛠️ Helm Override Default Values from `values.yaml`

> Learn how to override Helm chart values using `--set`, `-f myvalues.yaml`, inspect rendered manifests, and understand Helm’s value hierarchy.

---

## 📘 Step-01: Introduction

We will learn the following in this section:

* `helm install --set`
* `helm upgrade -f myvalues.yaml`
* `--dry-run`
* `--debug`
* `helm get values`
* `helm get values --revision`
* `helm get manifest`
* `helm get manifest --revision`
* `helm get all`

### 📚 Also covered:

* **Values Hierarchy** and
* **Deleting default keys** by assigning `null`

---

## 🔁 Step-02: Override default NodePort `31231` with `--set`

### 📝 Step-02-01: Review `mychart1` Helm Chart `values.yaml`

🔗 [mychart1 values.yaml](https://github.com/stacksimplify/helm-charts/blob/main/mychart1/values.yaml)

---

### 🧪 Step-02-02: Learn `--dry-run` and `--debug` flags

Install Helm Chart by overriding NodePort `31231` with `31240`:

```bash
# Dry-run install with a value override
helm install myapp901 stacksimplify/mychart1 \
  --set service.nodePort=31240 \
  --dry-run

# Dry-run with debug output
helm install myapp901 stacksimplify/mychart1 \
  --set service.nodePort=31240 \
  --dry-run --debug
```

### ✅ Sample Output with `--debug`:

```yaml
NAME: myapp901
NAMESPACE: default
STATUS: pending-install
REVISION: 1

USER-SUPPLIED VALUES:
service:
  nodePort: 31240

COMPUTED VALUES:
fullnameOverride: ""
image:
  pullPolicy: IfNotPresent
  repository: ghcr.io/stacksimplify/kubenginx
  tag: ""
nameOverride: ""
podAnnotations: {}
replicaCount: 1
service:
  nodePort: 31240
  port: 80
  type: NodePort
```

---

### 🚀 Step-02-03: Install using `--set` and test

```bash
helm install myapp901 stacksimplify/mychart1 \
  --set service.nodePort=31240

# Check resources
helm status myapp901 --show-resources
```

🌐 **Access Application:**

```
http://localhost:31240
```

---

## 📄 Step-03: Review `myvalues.yaml`

**Location**: `09-Helm-Override-Values/myvalues.yaml`

```yaml
# Change-1: change replicas from 1 to 2
replicaCount: 2

# Change-2: Override app version to 2.0.0
image:
  repository: ghcr.io/stacksimplify/kubenginx
  pullPolicy: IfNotPresent
  tag: "2.0.0"

# Change-3: Change nodePort from 31240 to 31250
service:
  type: NodePort
  port: 80
  nodePort: 31250
```

---

## 🔁 Step-04: Upgrade with `-f myvalues.yaml`

We can use either `-f myvalues.yaml` or `--values myvalues.yaml`.

```bash
# Navigate to correct directory
cd 09-Helm-Override-Values

# Validate with dry-run and debug
helm upgrade myapp901 stacksimplify/mychart1 \
  -f myvalues.yaml \
  --dry-run --debug

# Perform upgrade
helm upgrade myapp901 stacksimplify/mychart1 -f myvalues.yaml

# Check status and resources
helm status myapp901 --show-resources
```

### ✅ Observations:

1. Two Pods should be running (`replicaCount: 2`)
2. NodePort changes to `31250`

🌐 **Access Application:**

```
http://localhost:31250
```

✅ **Expected UI/Response**: V2 of the application (based on `tag: 2.0.0`)

---

## 📥 Step-05: `helm get values` command

Download a values file from the current release:

```bash
# Current values of the latest revision
helm get values myapp901
```

📝 **Sample Output**:

```yaml
USER-SUPPLIED VALUES:
image:
  pullPolicy: IfNotPresent
  repository: ghcr.io/stacksimplify/kubenginx
  tag: 2.0.0
replicaCount: 2
service:
  nodePort: 31250
  port: 80
  type: NodePort
```

```bash
# Check revision history
helm history myapp901

# Get values from a specific revision
helm get values myapp901 --revision 1
```

📝 **Sample Revision 1 Output**:

```yaml
USER-SUPPLIED VALUES:
service:
  nodePort: 31240
```

---

## 📜 Step-06: `helm get manifest` command

Retrieve rendered Kubernetes manifests for a given release.

```bash
# Latest manifest
helm get manifest myapp901

# Manifest for a specific revision
helm get manifest myapp901 --revision 1
```

---

## 📚 Step-07: `helm get all` command

```bash
helm get all myapp901
```

> ✅ Displays:

* Notes
* Supplied values
* Rendered manifest
* Chart metadata
* Hook output (if any)

📝 `helm get notes` and `helm get hooks` can be explored later during chart development.

---

## 🧹 Step-08: Uninstall Helm Release

```bash
helm uninstall myapp901

# Confirm deletion
helm list
```

---

## 📊 Step-09: Values Hierarchy

Understanding how Helm merges values:

| Precedence  | Source                                  |
| ----------- | --------------------------------------- |
| 1 (lowest)  | Subchart `values.yaml`                  |
| 2           | Parent chart `values.yaml`              |
| 3           | User-supplied file (`-f myvalues.yaml`) |
| 4 (highest) | Inline override (`--set key=value`)     |

> 🔺 The higher the level, the more precedence it has.

---

## ❌ Step-10: Deleting a Default Key with `null`

### 🎯 Objective:

Remove a default key (`nodePort`) so Helm lets Kubernetes auto-assign it.

---

```bash
# Release: myapp901
helm install myapp901 stacksimplify/mychart1 --atomic
helm status myapp901 --show-resources
```

🌐 Access:

```
http://localhost:31231
```

---

```bash
# Option-1: Override with another valid port
helm install myapp902 stacksimplify/mychart1 \
  --set service.nodePort=31232
```

---

```bash
# Option-2: Delete the default key using null
helm install myapp902 stacksimplify/mychart1 \
  --set service.nodePort=null \
  --dry-run --debug

# Perform install
helm install myapp902 stacksimplify/mychart1 \
  --set service.nodePort=null
```

📝 **Notes**:
1. We will choose option-2 to demonstrate the concept "Deleting a default Key by passing null"
2. For NodePort Service, if we dont define the "nodePort" argument, it by default assigns a port dynamically from the port range 30000-32767.
3. In our case already 31231 is used, other than that port it will allocate someother port when we pass null.
4. In short, if we dont want to pass the default values present in values.yaml as-is, we dont need to change the complete chart with a new version, we can just pass null.


```bash
# Verify the assigned NodePort
kubectl get svc myapp902-mychart1
```

---

## 🧼 Final Cleanup

```bash
helm uninstall myapp901
helm uninstall myapp902
helm list
```

---