# Helm Dependency - Import Values Implicit

## Step-01: Introduction

Helm allows **implicit import** of values from subcharts into the parent chart using the `import-values` key. Unlike the explicit approach (which uses exported values), the implicit method directly maps specific keys from a subchart’s `values.yaml` into designated paths in the parent chart.

This is especially useful when you want to promote select parts of a subchart’s configuration into the parent without requiring an `exports` block.

---

## Step-02: Define Implicit Imports in Chart.yaml

**File:** `parentchart/Chart.yaml`

```yaml
apiVersion: v2
name: parentchart
description: Learn Helm Dependency Concepts
type: application
version: 0.1.0
appVersion: "1.16.0"
dependencies:
- name: mychart1
  version: "0.1.0"
  repository: "file://charts/mychart1"
  alias: childchart1
  tags: 
    - frontend

- name: mychart2
  version: "0.4.0"
  repository: "file://charts/mychart2"
  alias: childchart2
  tags: 
    - backend
  import-values:  # Implicit Values Use Case
    - child: service
      parent: mychart2service
    - child: image
      parent: mychart2image
```

* `child`: refers to the path inside `mychart2/values.yaml`
* `parent`: specifies the key under which this value will be available in the parent chart

---

## Step-03: Use Imported Values in a Template

**File:** `parentchart/templates/configmap.yaml`

```yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: {{ include "parentchart.fullname" . }}-import-implicit
data:
  serviceType: {{ .Values.mychart2service.type }}
  servicePort: {{ .Values.mychart2service.port | quote }}
  servicenodePort: {{ .Values.mychart2service.nodePort | quote }}
  imageRepository: {{ .Values.mychart2image.repository }}
```

* This configmap demonstrates how the implicitly imported values from the subchart can be consumed in the parent chart.

---

## Step-04: Install and Verify

```bash
# Navigate to the chart directory
cd parentchart

# Install the Helm chart
helm install myapp1 . --atomic

# Check Helm release
helm list
helm status myapp1 --show-resources

# Kubernetes resources
kubectl get deploy
kubectl get pods
kubectl get cm
kubectl get cm myapp1-parentchart-import-implicit -o yaml
```

### 🔍 Expected Output:

```yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: myapp1-parentchart-import-implicit
data:
  serviceType: NodePort
  servicePort: "80"
  servicenodePort: "31232"
  imageRepository: ghcr.io/stacksimplify/kubenginx
```

This confirms that the values from `mychart2/values.yaml` were successfully imported and accessed through `mychart2service` and `mychart2image` in the parent chart.

---

## Step-05: Disable Subchart and Test

```bash
# Install the chart with backend subchart disabled
helm install myapp1 . --atomic --set tags.backend=false
```

### 🔥 Expected Error:

```shell
Error: INSTALLATION FAILED: template: parentchart/templates/configmap.yaml:6:25: executing "parentchart/templates/configmap.yaml" at <.Values.mychart2service.type>: nil pointer evaluating interface {}.type
```

### 🧠 Why the error?

* The subchart `mychart2` is not rendered, so its values (which were meant to be imported) do not exist.
* Because Helm still tries to render the parent template using those now-missing values, it fails.

---

## Resolution Tips

* You can guard template rendering with an `if` block to avoid runtime errors when subcharts are conditionally disabled:

```yaml
{{- if .Values.mychart2service }}
apiVersion: v1
kind: ConfigMap
metadata:
  name: {{ include "parentchart.fullname" . }}-import-implicit
data:
  serviceType: {{ .Values.mychart2service.type }}
  servicePort: {{ .Values.mychart2service.port | quote }}
  servicenodePort: {{ .Values.mychart2service.nodePort | quote }}
  imageRepository: {{ .Values.mychart2image.repository }}
{{- end }}
```

---

## Step-06: Cleanup

```bash
helm uninstall myapp1
```

---

## Summary

* **Import Values (Implicit)** allows you to map values from a subchart to specific paths in the parent without needing an `exports` block.
* It simplifies configuration but requires careful handling when the subchart is optional.
* Always guard templates using imported values to avoid rendering errors when subcharts are disabled.
