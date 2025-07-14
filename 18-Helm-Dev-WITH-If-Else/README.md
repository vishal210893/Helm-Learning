# 🚦 Helm Development – Flow Control `if/else` with Nested Boolean Logic and `with` Block

---

## 📘 Step-01: Introduction

In Helm, the `if`, `else if`, and `else` constructs allow you to introduce conditional logic into your templates. These are essential when building dynamic manifests that must behave differently based on input values such as environment, features, or flags.

Additionally, Helm implements all logical operators (`eq`, `ne`, `and`, `or`, `not`, etc.) as **functions** instead of traditional symbols. This means you'll use them like `eq .foo "bar"` instead of `.foo == "bar"`.

### 🧠 Key Highlights:

* Logical flow in Helm is **function-based**: `eq`, `and`, `or`, `not`, etc.
* `with` block lets you scope into a nested structure to avoid repeating long paths.
* You can **combine `if` and `with`** to simplify deeply nested logic.

### 🔗 Additional Reference:

👉 [Operators Are Functions – Helm Docs](https://helm.sh/docs/chart_template_guide/functions_and_pipelines/#operators-are-functions)

---

## 🧾 Step-02: Sample `values.yaml`

```yaml
myapp:
  env: prod
  retail:
    enableFeature: true
```

This structure allows us to conditionally change the number of replicas based on:

* Whether the environment is `prod`, `qa`, or something else.
* Whether a retail-specific feature is enabled.

---

## 🔁 Step-03: Logic and Flow Control Function: `and`

### 📙 Function Signature

```gotemplate
and .Arg1 .Arg2 ...
```

### 💡 Use Case

The `and` function returns `true` **only if all arguments evaluate to true/non-empty**. If any value is falsy (like `false`, `""`, `0`, or `nil`), the entire result is false.

---

## 🛠️ Step-04: Template with `if/else`, Boolean Check and `with`

Here’s how you build nested flow control using the `and` function and `with` block to cleanly reference `.Values.myapp`:

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: {{ .Release.Name }}-{{ .Chart.Name }}
  labels:
    app: nginx
spec:
{{- with .Values.myapp }}
{{- if and .retail.enableFeature (eq .env "prod") }}
  replicas: 6
{{- else if eq .env "prod" }}
  replicas: 4
{{- else if eq .env "qa" }}  
  replicas: 2
{{- else }}  
  replicas: 1  
{{- end }}
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

### 📌 Explanation

* `with .Values.myapp`: Sets the current context (`.`) to `myapp`, so we don’t need to repeatedly write `.Values.myapp`.
* `and .retail.enableFeature (eq .env "prod")`: Only renders 6 replicas **if both conditions** are true.
* Else, it checks whether the environment is prod or qa and adjusts the replicas accordingly.
* Defaults to 1 replica for any other case (like `dev`, `staging`, or if unset).

---

## ✅ Step-05: Testing and Verifying Helm Logic

### 🔧 Test Different Scenarios

```bash
cd helmbasics

# Test when enableFeature=true and env=prod → replicas = 6
helm template myapp1 . --set myapp.retail.enableFeature=true

# Test when enableFeature=false and env=prod → replicas = 4
helm template myapp1 . --set myapp.retail.enableFeature=false

# Test for qa → replicas = 2
helm template myapp1 . --set myapp.env=qa

# Test for dev or unknown → replicas = 1
helm template myapp1 . --set myapp.env=dev
```

### 🧪 Simulate Installation

```bash
# Dry-run to preview manifest rendering
helm install myapp1 . --dry-run

# Real install
helm install myapp1 . --atomic
```

### 🔍 Inspect and Validate

```bash
# Check resources created (e.g., number of pods)
helm status myapp1 --show-resources

# Clean up
helm uninstall myapp1
```

---

## 📌 Tips & Best Practices

| Pattern           | Description                                                          |
| ----------------- | -------------------------------------------------------------------- |
| `with` + `if`     | Great for scoping into nested objects and applying conditional logic |
| `and`             | Only true if **all conditions** are true                             |
| `eq`              | Helm’s way to check equality (e.g., `eq .foo "bar"`)                 |
| `$.Values` or `$` | Use `$` to access global scope if you're inside a nested `with`      |

---

## 📚 Further Learning

* [Helm `if`/`else` Control Structures](https://helm.sh/docs/chart_template_guide/control_structures/#ifelse)
* [Helm Function Reference (Logic)](https://helm.sh/docs/chart_template_guide/function_list/#logic-and-flow-control-functions)
* [Gotemplate Pipelines](https://pkg.go.dev/text/template)

---