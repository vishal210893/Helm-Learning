# 🧠 Helm Development – Flow Control `if/else` with `or` Function

---

## 📘 Step-01: Introduction

In Helm templates, you can use conditional logic to render specific Kubernetes configurations depending on values supplied in `values.yaml` or via `--set`. One of the most commonly used logical functions in real-world deployments is the `or` function, which allows you to trigger blocks if **any** one of the multiple conditions is met.

### 🔍 Why `or` Function Matters:

* Useful when multiple values should produce the same outcome (e.g., environments `prod` or `uat`)
* Helps to reduce repeated `if` statements
* Keeps your templates readable and concise

---

### 🧠 Operators Are Functions in Helm

Helm uses Go template syntax, so logic like `eq`, `and`, `or`, `ne` are implemented as **functions** instead of traditional operators.

📌 Example:

```gotemplate
{{ if or (eq .Values.env "prod") (eq .Values.env "uat") }}
  # Will be executed if env is prod or uat
{{ end }}
```

🔗 [Reference: Operators Are Functions](https://helm.sh/docs/chart_template_guide/functions_and_pipelines/#operators-are-functions)

---

## 🧾 IF-ELSE Syntax in Helm

```gotemplate
{{ if PIPELINE }}
  # Do something
{{ else if OTHER PIPELINE }}
  # Do something else
{{ else }}
  # Default case
{{ end }}
```

This allows branching logic based on provided input. You can also chain logic functions for combined conditions.

---

## 🧾 Step-02: Review Sample `values.yaml`

Here’s an example config defining an environment:

```yaml
myapp:
  env: prod
```

You can override this with `--set myapp.env=qa` or a custom values file.

---

## ⚙️ Step-03: Logic Function – `or`

Helm’s `or` function returns the **first non-empty/true** argument from its input list.

### 🔹 Syntax:

```gotemplate
{{ if or .Arg1 .Arg2 .Arg3 }}
  # Executes if any of Arg1, Arg2, or Arg3 evaluates to true
{{ end }}
```

🔗 [Helm Logic Functions Reference](https://helm.sh/docs/chart_template_guide/function_list/#logic-and-flow-control-functions)

---

## 🧩 Step-04: Implement Conditional `replicas` Count Using `or`

Here’s a full working example of a Kubernetes Deployment YAML with conditional `replicas` based on `env`:

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: {{ .Release.Name }}-{{ .Chart.Name }}
  labels:
    app: nginx
spec:
{{- if or (eq .Values.myapp.env "prod") (eq .Values.myapp.env "uat") }}
  replicas: 6
{{- else if eq .Values.myapp.env "qa" }}  
  replicas: 2
{{- else }}  
  replicas: 1  
{{- end }}
  selector:
    matchLabels:
      app: nginx
  template:
    metadata:
      labels:
        app: nginx
    spec:
      containers:
      - name: nginx
        image: ghcr.io/stacksimplify/kubenginx:4.0.0
        ports:
        - containerPort: 80
```

### 📝 Explanation:

* If `env` is `prod` or `uat` → **6 replicas**
* If `env` is `qa` → **2 replicas**
* For any other value or unset → **1 replica**

---

## ✅ Step-05: Test Output with `helm template`

```bash
cd helmbasics
```

### 🔹 Test with `env=prod`

```bash
helm template myapp1 . --set myapp.env=prod
```

### 🔹 Test with `env=uat`

```bash
helm template myapp1 . --set myapp.env=uat
```

### 🔹 Test with `env=dev`

```bash
helm template myapp1 . --set myapp.env=dev
```

### 🔹 Test with `env` unset or null

```bash
helm template myapp1 . --set myapp.env=null
```

---

## 🚀 Deploy and Validate

### 🧪 Dry-run Installation

```bash
helm install myapp1 . --dry-run
```

### 🟢 Real Install

```bash
helm install myapp1 . --atomic
```

### 🔍 Inspect Kubernetes Resources

```bash
helm status myapp1 --show-resources
```

---

## 🧹 Clean Up

```bash
helm uninstall myapp1
```

---

## 🧠 Summary Table – Replica Logic

| Environment | `replicas` | Helm Command                             |
| ----------- | ---------- | ---------------------------------------- |
| prod        | 6          | `--set myapp.env=prod`                   |
| uat         | 6          | `--set myapp.env=uat`                    |
| qa          | 2          | `--set myapp.env=qa`                     |
| dev/null    | 1          | `--set myapp.env=dev` or no override set |

---

## 📚 Additional Resources

* 🔗 [Helm Flow Control](https://helm.sh/docs/chart_template_guide/control_structures/)
* 🔗 [Function Reference – `or`](https://helm.sh/docs/chart_template_guide/function_list/#or)
* 🔗 [Template Debugging with `helm template`](https://helm.sh/docs/helm/helm_template/)

---
