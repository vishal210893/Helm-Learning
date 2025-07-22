# Helm Dependency - Import Values Explicit

## Step-01: Introduction

Helm supports importing values from subcharts into the parent chart using the `import-values` key. This feature is particularly helpful when subcharts expose specific configuration (via `exports`) that the parent chart needs to use directly.

This approach enables better encapsulation and reuse while still offering a way for the parent chart to integrate deeply with the subchart's exported configuration.

---

## Step-02: Define Exported Values in Subchart

**File:** `parentchart/charts/mychart1/values.yaml`

```yaml
# Export Values - mychart1 (Used for Import Values Explicit Use Case)
exports:
  mychart1Data:
    mychart1appInfo:
      appName: kapp1
      appType: MicroService
      appDescription: Used for listing products    
```

* The `exports` field defines data that can be exposed to the parent.
* This structure is key to enabling value promotion via `import-values`.

---

## Step-03: Configure Dependency with `import-values`

**File:** `parentchart/Chart.yaml`

```yaml
dependencies:
- name: mychart1
  version: "0.1.0"
  repository: "file://charts/mychart1"
  alias: childchart1
  tags: 
    - frontend
  import-values:
    - mychart1Data  # Explicit Values Import Use Case
```

* Here, Helm will import values from `exports.mychart1Data` into the parent scope.

---

## Step-04: Use Imported Values in Parent Chart Template

**File:** `parentchart/templates/configmap.yaml`

```yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name:  {{ include "parentchart.fullname" . }}-import-explicit
data:
{{- toYaml .Values.mychart1appInfo | nindent 2 }}
```

> After import, the value `mychart1appInfo` will be directly accessible in the parent chart at `.Values.mychart1appInfo`.

---

## Step-05: Deploy and Verify

```bash
# Change to Chart Directory
cd parentchart

# Helm Install
helm install myapp1 . --atomic

# Verify Helm Release
helm list
helm status myapp1 --show-resources

# Check Kubernetes resources
kubectl get deploy
kubectl get pods
kubectl get cm

# Review ConfigMap
kubectl get cm myapp1-parentchart-import-explicit -o yaml
```

### 🔍 Expected ConfigMap Output:

```yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: myapp1-parentchart-import-explicit
data:
  appName: kapp1
  appType: MicroService
  appDescription: Used for listing products
```

> This confirms that the values exported by the subchart have been successfully imported and rendered in the parent chart.

---

## Step-06: Test When Subchart is Disabled

```bash
# Disable subchart by turning off its tag
helm install myapp1 . --atomic --set tags.frontend=false

# Check ConfigMap
kubectl get cm myapp1-parentchart-import-explicit -o yaml
```

### 🔍 Observation:

* The config map should either not exist or have no `.data` field since the subchart was disabled and its exports were never imported.

---

## Step-07: Cleanup

```bash
# Uninstall release
helm uninstall myapp1
```

---

## Summary

* Use `exports` in a subchart to explicitly define what values should be shared.
* Use `import-values` in the parent chart’s dependency section to pull in those values.
* This keeps charts modular and avoids deep coupling, while still allowing value reuse across chart layers.

This explicit approach to value import enhances clarity and control over what is shared between charts.
