# 🧭 Helm Development – Flow Control Using `with` Action

---

## 📘 Step-01: Introduction

Helm’s templating engine is designed not only for rendering YAML but also for creating reusable, scoped, and manageable templates. One of the key flow control constructs in Helm is the `with` action.

The `with` statement allows us to:

* Narrow down the **scope** (dot `.`) to a specific object in the chart values.
* Write **cleaner and more readable** templates when accessing nested values repeatedly.
* Avoid verbosity by eliminating repeated access paths like `.Values.foo.bar.baz`.

### 🔹 Syntax:

```gotemplate
{{ with PIPELINE }}
  # Inside this block, "." refers to the result of PIPELINE
{{ end }}
```

---

## 🧾 Step-02: Sample `values.yaml`

This file provides annotations to be inserted into a pod spec:

```yaml
podAnnotations: 
  appName: myapp1
  appType: webserver
  appTech: HTML
```

---

## ⚙️ Step-03: Using the `with` Block

Let’s say we want to apply the annotations from `.Values.podAnnotations` into a pod spec. Using `with`, we can simplify access:

```yaml
template:
  metadata:
    {{- with .Values.podAnnotations }}
    annotations:
      {{- toYaml . | nindent 8 }}        
    {{- end }}
```

### ✅ Explanation:

* `.Values.podAnnotations` is passed into the `with` block.
* Inside the block, `.` becomes `.Values.podAnnotations`.
* We use `toYaml` to render the key-value pairs correctly.
* `nindent 8` ensures proper YAML indentation under the `annotations:` field.

---

## 🔍 Step-04: Test the `with` Block Implementation

```bash
# Navigate to Chart Directory
cd helmbasics

# Render manifest using template
helm template myapp101 .

# Simulate install
helm install myapp101 . --dry-run
```

### 🧾 Expected Output

```yaml
annotations:
  appName: myapp1
  appTech: HTML
  appType: webserver
```

---

## ❌ Step-05: Trying to Access a Root Object from Inside `with` (Fails)

What if we try to use `.Release.Service` (a top-level object) **inside** the `with` block?

```yaml
template:
  metadata:
    {{- with .Values.podAnnotations }}
    annotations:
      {{- toYaml . | nindent 8 }}
      appManagedBy: {{ .Release.Service }}   # ❌ this will fail
    {{- end }}
```

### 🧨 Error:

```text
Error: template: helmbasics/templates/deployment.yaml:23:33:
executing "helmbasics/templates/deployment.yaml" at <.Release.Service>:
nil pointer evaluating interface {}.Service
```

### 🧠 Why It Fails:

Within the `with` block, `.` is scoped only to `.Values.podAnnotations`. The `.Release.Service` key does not exist within that object.

---

## ✅ Step-06: Fix Scope Access Using `$`

To reference the **global context** (the root object) within a nested block, use `$`:

```yaml
template:
  metadata:
    {{- with .Values.podAnnotations }}
    annotations:
      {{- toYaml . | nindent 8 }}
      appManagedBy: {{ $.Release.Service }}   # ✅ This works
    {{- end }}
```

### ✅ Expected Output:

```yaml
annotations:
  appName: myapp1
  appTech: HTML
  appType: webserver
  appManagedBy: Helm
```

---

## 🧠 Step-07: Advanced Scope – Nested Objects

Let’s now explore how to access specific keys from a deeply nested object using `with`.

### Sample `values.yaml`

```yaml
myapps:
  data: 
    config: 
      appName: myapp1
      appType: webserver
      appTech: HTML
      appDb: mysql
```

### Rendered in a ConfigMap Using `with`

```yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: {{ .Release.Name }}-{{ .Chart.Name }}
data: 
{{- with .Values.myapps.data.config }}
  application-name: {{ .appName }}
  application-type: {{ .appType }}
{{- end }}
```

### 🧾 Explanation:

* The `with` block resets scope to `.Values.myapps.data.config`.
* Now `.appName` and `.appType` are directly accessible.
* Clean and avoids long dotted paths like `.Values.myapps.data.config.appName`.

---

## 🧪 Step-08: Test Advanced `with` Scope

```bash
cd helmbasics

# Render the chart and confirm the ConfigMap values
helm template myapp101 .

# Optional dry-run install
helm install myapp101 . --dry-run
```

### 🧾 Expected Output:

```yaml
data:
  application-name: myapp1
  application-type: webserver
```

---

## ✅ Summary: When and Why to Use `with`

| Use Case                                           | Why Use `with`                           |
| -------------------------------------------------- | ---------------------------------------- |
| Deeply nested structures                           | Avoid redundant dotted paths             |
| Conditional rendering                              | Prevents clutter with scoped logic       |
| Repeated reference to same object                  | Improves clarity and reduces verbosity   |
| Need to access global values within a scoped block | Use `$` to break out of scoped dot (`.`) |

---

## 📚 Additional Resources

* 🔗 [Helm Template Guide: `with`](https://helm.sh/docs/chart_template_guide/control_structures/#the-with-statement)
* 🔗 [Function List: toYaml, nindent](https://helm.sh/docs/chart_template_guide/function_list/)
* 🔗 [Flow Control with Root Access via `$`](https://helm.sh/docs/chart_template_guide/function_list/#using-the-root-context)

---