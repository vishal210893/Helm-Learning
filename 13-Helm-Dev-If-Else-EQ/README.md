# 🧠 Helm Development – Flow Control with `if/else`

> In this guide, you’ll learn how to use conditional logic in Helm templates using `if`, `else if`, `else`, and Helm’s built-in logic functions like `eq`, `and`, `or`, and more.

---

## 📘 Step-01: Introduction

Helm's templating engine gives us access to Go's robust conditional logic. This allows us to build flexible, reusable charts that can behave differently based on user-provided values or environment settings.

### 🔑 Key Concepts:

* `if`, `else if`, and `else` help define conditional logic
* Operators like `eq`, `ne`, `and`, `or` are implemented as **functions**
* Helm conditionals are **function-based**, unlike typical programming languages

🔗 [Operators are Functions – Helm Docs](https://helm.sh/docs/chart_template_guide/functions_and_pipelines/#operators-are-functions)

---

### 📚 Syntax of `if/else` in Helm Templates

```gotemplate
{{ if PIPELINE }}
  # Do something
{{ else if OTHER PIPELINE }}
  # Do something else
{{ else }}
  # Default case
{{ end }}
```

> You can nest or chain conditions with `eq`, `and`, or `or` for richer expressions.

---

## 📄 Step-02: Review `values.yaml`

```yaml
# Set the environment to control behavior
myapp:
  env: prod
```

This value will determine which replica count is rendered in the template.

---

## ⚙️ Step-03: Logic and Flow Control – `eq`, `and`, `or`

Helm uses Go template functions for logic control. For example:

```gotemplate
{{- if eq .Values.myapp.env "prod" }}
```

* `eq`: Compares two values for equality
* `and`: Logical AND of multiple expressions
* `or`: Logical OR of multiple expressions

🔗 [Flow Control Functions – Helm Docs](https://helm.sh/docs/chart_template_guide/function_list/#logic-and-flow-control-functions)

---

## 🧩 Step-04: Apply `if/else` for Replica Count

Here’s how you conditionally define the number of replicas in a Deployment based on the environment:

```yaml
# templates/deployment.yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: {{ .Release.Name }}-{{ .Chart.Name }}
  labels:
    app: nginx
spec:
{{- if eq .Values.myapp.env "prod" }}
  replicas: 4 
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

> ✅ This logic ensures your chart scales differently in each environment without duplicating the chart or hardcoding values.

---

## 🔍 Step-05: Verify `if/else` Conditions

Use Helm CLI to simulate how values affect template rendering:

```bash
# Navigate to chart directory
cd helmbasics
```

### 🔹 Case 1: env = prod (From values.yaml)

```bash
helm template myapp1 .
```

✅ **Expected:**
`replicas: 4`

---

### 🔹 Case 2: env = qa (Override via CLI)

```bash
helm template myapp1 . --set myapp.env=qa
```

✅ **Expected:**
`replicas: 2`

---

### 🔹 Case 3: env = dev or unset (Default to else block)

```bash
helm template myapp1 . --set myapp.env=dev
```

✅ **Expected:**
`replicas: 1`

> 🧠 Any value not explicitly handled by `if` or `else if` will fall into the `else` case.

---

### 🚀 Deploy and Validate

```bash
# Dry-run installation (renders and validates)
helm install myapp1 . --dry-run

# Perform actual install
helm install myapp1 . --atomic
```

---

### 📋 Check Resources

```bash
# View Helm status and deployed resources
helm status myapp1 --show-resources
```

---

### 🧹 Clean Up

```bash
# Uninstall the release
helm uninstall myapp1
```

---

## ✅ Summary

| Condition                      | Command Example                              | Expected Output |
| ------------------------------ | -------------------------------------------- | --------------- |
| `env: prod` (from values.yaml) | `helm template myapp1 .`                     | `replicas: 4`   |
| `env: qa`                      | `helm template myapp1 . --set myapp.env=qa`  | `replicas: 2`   |
| `env: dev` or missing          | `helm template myapp1 . --set myapp.env=dev` | `replicas: 1`   |

---

## 📚 Additional References

* 🔗 [Operators as Functions](https://helm.sh/docs/chart_template_guide/functions_and_pipelines/#operators-are-functions)
* 🔗 [Logic and Flow Functions](https://helm.sh/docs/chart_template_guide/function_list/#logic-and-flow-control-functions)
* 🔗 [Go Templates for Helm](https://helm.sh/docs/chart_template_guide/templates_intro/)

---
