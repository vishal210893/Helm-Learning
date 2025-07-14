# 🧠 Helm Development – Flow Control `if/else` with Boolean Check and `and` Function

> Use Helm's built-in logic and flow control functions to conditionally render Kubernetes resources based on multiple flags like `env` and feature toggles.

---

## 📘 Step-01: Introduction

Helm’s templating engine offers full support for logical conditionals using Go template syntax. This allows for **declarative branching** inside your templates to support multiple deployment modes like `dev`, `qa`, or `prod`.

### Why Use Conditional Logic in Charts?

* Enable or disable parts of the template based on user-defined settings
* Customize resource counts or behavior across environments
* Control feature flags and toggles dynamically

---

### 🔍 Operators as Functions

In Helm, operators like `eq`, `ne`, `lt`, `gt`, `and`, and `or` are implemented as functions.

📌 In a conditional block, you can use:

```gotemplate
{{ if eq .Values.myapp.env "prod" }}
  # Do something
{{ else }}
  # Do something else
{{ end }}
```

🔗 [Reference: Operators are Functions – Helm Docs](https://helm.sh/docs/chart_template_guide/functions_and_pipelines/#operators-are-functions)

---

## 🧾 IF-ELSE Syntax Refresher

```gotemplate
{{ if PIPELINE }}
  # Do something
{{ else if OTHER PIPELINE }}
  # Do something else
{{ else }}
  # Default case
{{ end }}
```

> You can chain multiple expressions, use nested `if` blocks, or combine logic using `and`, `or`, and comparison operators.

---

## 📄 Step-02: Sample `values.yaml` with Boolean and Env

```yaml
# values.yaml
myapp:
  env: prod
  retail:
    enableFeature: true
```

This example introduces a **feature flag** (`enableFeature`) under `myapp.retail`, in combination with an environment flag.

---

## ⚙️ Step-03: Logic Function – `and`

The `and` function returns the **boolean AND** of two or more arguments. It is commonly used to **combine multiple conditions** in Helm templates.

### 🔹 Syntax:

```gotemplate
{{ if and .Values.flagA (eq .Values.env "prod") }}
  # Do something only if BOTH conditions are true
{{ end }}
```

🔗 [Logic Functions – Helm Docs](https://helm.sh/docs/chart_template_guide/function_list/#logic-and-flow-control-functions)

---

## 🧩 Step-04: Implement `if/else` for Replicas with Boolean and Env

```yaml
# templates/deployment.yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: {{ .Release.Name }}-{{ .Chart.Name }}
  labels:
    app: nginx
spec:
{{- if and .Values.myapp.retail.enableFeature (eq .Values.myapp.env "prod") }}
  replicas: 6
{{- else if eq .Values.myapp.env "prod" }}
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

### 📝 Explanation:

* If `env` is `prod` **and** feature flag is enabled → **6 replicas**
* If only `env` is `prod` → **4 replicas**
* If `env` is `qa` → **2 replicas**
* Otherwise → **1 replica**

This structure provides **flexibility** to model behavior for both environment-based scaling and feature-based toggles.

---

## ✅ Step-05: Verify Output with `helm template`

```bash
cd helmbasics
```

### 🔹 Case 1: Feature enabled + env=prod → replicas: 6

```bash
helm template myapp1 . --set myapp.retail.enableFeature=true
```

### 🔹 Case 2: Feature disabled + env=prod → replicas: 4

```bash
helm template myapp1 . --set myapp.retail.enableFeature=false
```

### 🔹 Case 3: env=qa → replicas: 2

```bash
helm template myapp1 . --set myapp.env=qa
```

### 🔹 Case 4: env=dev (or anything else) → replicas: 1

```bash
helm template myapp1 . --set myapp.env=dev
```

---

## 🚀 Install the Chart

### 🔄 Dry-run

```bash
helm install myapp1 . --set myapp.retail.enableFeature=true --dry-run
```

### 🟢 Real install

```bash
helm install myapp1 . --set myapp.retail.enableFeature=true --atomic
```

---

## 🔍 Verify Resources

```bash
helm status myapp1 --show-resources
```

> Confirm that the number of replicas matches the expected condition.

---

## 🧹 Clean Up

```bash
helm uninstall myapp1
```

---

## 🧠 Summary Table

| Feature Enabled | Environment | Command Example                          | Expected Replicas |
| --------------- | ----------- | ---------------------------------------- | ----------------- |
| ✅ true          | prod        | `--set myapp.retail.enableFeature=true`  | 6                 |
| ❌ false         | prod        | `--set myapp.retail.enableFeature=false` | 4                 |
| ✅/❌ any value   | qa          | `--set myapp.env=qa`                     | 2                 |
| ✅/❌ any value   | dev/other   | `--set myapp.env=dev`                    | 1                 |

---

## 📚 Additional Learning Resources

* 🔗 [Helm Operators Are Functions](https://helm.sh/docs/chart_template_guide/functions_and_pipelines/#operators-are-functions)
* 🔗 [Logic & Flow Control Functions](https://helm.sh/docs/chart_template_guide/function_list/#logic-and-flow-control-functions)
* 🔗 [Go Template Syntax Reference](https://pkg.go.dev/text/template)

---