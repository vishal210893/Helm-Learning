# 🔁 Helm Development – Flow Control: `if/else` with `not` Function

---

## 📘 Step-01: Introduction

Helm templating allows us to build flexible and dynamic Kubernetes manifests using control flow logic such as `if/else`. When combined with logical functions like `not`, `eq`, `and`, and `or`, we can fine-tune when and how resources get rendered—based on values from `values.yaml` or runtime inputs.

This is especially powerful in multi-environment deployments (e.g., dev, qa, prod) where configuration needs vary significantly.

### 🧠 Operators Are Functions in Helm

Unlike imperative languages, Helm uses Go template syntax, where logic operators are implemented as functions.

Examples:

* `eq` = equal to
* `not` = logical NOT
* `and`, `or` = boolean logic
* `ne`, `lt`, `gt` = relational functions

🔗 [Reference: Operators Are Functions](https://helm.sh/docs/chart_template_guide/functions_and_pipelines/#operators-are-functions)

---

### 🧾 IF-ELSE Template Syntax

```gotemplate
{{ if PIPELINE }}
  # Rendered if pipeline evaluates to true
{{ else if OTHER PIPELINE }}
  # Rendered if above condition fails and this is true
{{ else }}
  # Rendered if all conditions fail
{{ end }}
```

You can nest `if`, `else if`, and `else` blocks inside templates for conditional resource rendering.

---

## 🧾 Step-02: Sample `values.yaml`

We’ll define an environment variable in `values.yaml` as our control input:

```yaml
myapp:
  env: prod
```

You can override this at install/upgrade time with:

```bash
--set myapp.env=dev
```

---

## 🧠 Step-03: Logic and Flow Control Function – `not`

The `not` function is used to **negate** a given boolean expression. It returns:

* `true` if the input is falsey
* `false` if the input is truthy

### 🔹 Syntax:

```gotemplate
{{ not .Values.flag }}
{{ not (eq .Values.env "prod") }}
```

🔗 [Helm Logic Functions – not](https://helm.sh/docs/chart_template_guide/function_list/#not)

---

## 🧩 Step-04: Implement Conditional `replicas` Count Using `not`

This example renders a different `replicaCount` based on whether the environment is **not** production. If it's **not prod**, we set replicas to 1; otherwise, we scale to 6.

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: {{ .Release.Name }}-{{ .Chart.Name }}
  labels:
    app: nginx
spec:
{{- if not (eq .Values.myapp.env "prod") }}
  replicas: 1
{{- else }}  
  replicas: 6
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

* When `.Values.myapp.env != "prod"` → use **1 replica**
* When `.Values.myapp.env == "prod"` → use **6 replicas**

This logic makes the chart automatically scale down in non-production environments like `dev`, `qa`, or when `env` is unset.

---

## ✅ Step-05: Testing with `helm template` and `install`

### 📂 Navigate to Chart Directory

```bash
cd helmbasics
```

### 🔍 Test with Different Environments Using `helm template`

```bash
# Case 1: env=prod → 6 replicas
helm template myapp1 . --set myapp.env=prod

# Case 2: env=dev → 1 replica
helm template myapp1 . --set myapp.env=dev

# Case 3: env not set → 1 replica (since eq "null" != "prod")
helm template myapp1 . --set myapp.env=null
```

### 🧪 Dry Run Installation

```bash
helm install myapp1 . --dry-run
```

### 🚀 Real Installation

```bash
helm install myapp1 . --atomic
```

### 🔍 Inspect Running Resources

```bash
helm status myapp1 --show-resources
```

### 🧹 Uninstall Release

```bash
helm uninstall myapp1
```

---

## 📚 Summary Table

| Environment | Condition                 | Replicas |
| ----------- | ------------------------- | -------- |
| prod        | `not (eq "prod")` → false | 6        |
| dev         | `not (eq "prod")` → true  | 1        |
| uat         | true                      | 1        |
| null/unset  | true                      | 1        |

---

## 📚 Additional Resources

* 🔗 [Helm Control Structures](https://helm.sh/docs/chart_template_guide/control_structures/)
* 🔗 [Helm Function List – Logic](https://helm.sh/docs/chart_template_guide/function_list/#logic-and-flow-control-functions)
* 🔗 [Helm Template Debugging](https://helm.sh/docs/helm/helm_template/)

---