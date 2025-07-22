# Helm Development - Flow Control Range with Dictionary

## Step-01: Introduction

* The `range` action can also be used to iterate over dictionaries (maps) in Helm templates.
* When ranging over maps, Helm provides both the key and value variables (e.g., `range $key, $value := ...`).
* This technique is useful for dynamically building structured data like `ConfigMaps` with arbitrary key-value pairs.

---

## Step-02: Range with Key-Value Pairs (Map/Dictionary)

* **Goal:** Generate a `ConfigMap` from a dictionary object defined in `values.yaml`
* **Input Source:** `backupfiles/namespace.yaml`
* **Output File:** `helmbasics/templates/namespace.yaml`

```yaml
# values.yaml
myapps:
  config1: 
    appName: myapp1
    appType: webserver
    appTech: HTML
    appDb: mysql
  config2: 
    appName: myapp2
    appType: webserver
    appTech: HTML
    appDb: mysql
```

```yaml
# helmbasics/templates/namespace.yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: {{ .Release.Name }}-{{ .Chart.Name }}-configmap1
data:
{{- range $key, $value := .Values.myapps.config1 }}
  {{ $key }}: {{ $value }}
{{- end }}
```

### Test Commands

```bash
cd helmbasics

# Validate template rendering
helm template myapp1 .

# Simulate Helm install
helm install myapp1 . --dry-run

# Real install
helm install myapp1 . --atomic

# Check Helm and Kubernetes state
helm list
helm status myapp1 --show-resources
kubectl get configmap
kubectl get configmap myapp1-helmbasics-configmap1 -o yaml

# Clean up
helm uninstall myapp1
```

**Notes:**

* When ranging over a dictionary, `.Values.myapps.config1` is treated as a map. Each `key` and `value` is accessible inside the loop.
* The `nindent` or standard YAML indentation can be applied to align key-value output correctly.

---

## Step-03: Access Root Object Inside Range Using Helm Variable

* **Goal:** Enhance the previous example by appending the chart name (a Helm built-in object) to each value.
* **Input Source:** `backupfiles/namespace-with-variable.yaml`
* **Output File:** `helmbasics/templates/namespace-with-variable.yaml`

```yaml
# values.yaml
myapps:
  config1: 
    appName: myapp1
    appType: webserver
    appTech: HTML
    appDb: mysql
  config2: 
    appName: myapp2
    appType: webserver
    appTech: HTML
    appDb: mysql
```

```yaml
# helmbasics/templates/namespace-with-variable.yaml
{{- $chartName := .Chart.Name }}
apiVersion: v1
kind: ConfigMap
metadata:
  name: {{ .Release.Name }}-{{ .Chart.Name }}-configmap2
data:
{{- range $key, $value := .Values.myapps.config2 }}
  {{ $key }}: {{ $value }}-{{ $chartName }}
{{- end }}
```

### Test Commands

```bash
cd helmbasics

# Template render check
helm template myapp1 .

# Dry-run simulation
helm install myapp1 . --dry-run

# Install and inspect
helm install myapp1 . --atomic
helm list
kubectl get configmap
kubectl get configmap myapp1-helmbasics-configmap2 -o yaml

# Cleanup
helm uninstall myapp1
```

**Important Notes:**

* Defining `$chartName := .Chart.Name` at the top ensures it's available inside the `range` block.
* Without this, directly referencing `.Chart.Name` inside the range may not resolve as expected due to scope shadowing.
* You can use `$` to refer to the root scope, but assigning to a variable is cleaner and more readable when reused multiple times.
