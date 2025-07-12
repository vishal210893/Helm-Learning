# 🧩 Helm Template Functions and Pipelines

> Dive deep into Helm’s powerful template engine based on Go’s text/template. Learn how to use functions, pipelines, default values, YAML conversions, and whitespace control to build dynamic, maintainable Kubernetes manifests.

---

## 📘 Step-01: Introduction

Helm templates are **not static YAML files**. They are **rendered dynamically** using Go's templating engine, which allows us to inject values, apply transformations, and control whitespace or indentation.

This section covers the following:

1. **Template Actions** using `{{ }}`
2. **Action Elements** like `{{ .Release.Name }}`
3. Using the `quote` function
4. How to **chain transformations** using pipelines
5. Using the `default` and `lower` functions
6. How to **control whitespace** with `{{- -}}`
7. Using `indent` and `nindent` for structured YAML
8. Leveraging `toYaml` to convert complex values

---

## 🧠 Step-02: Template Actions (`{{ }}`)

### What is a Template Action?

In Helm, anything inside double curly braces (`{{ ... }}`) is known as a **template action**. These are parsed and executed by Helm's engine during template rendering.

* Action Elements like `{{ .Chart.Name }}` are dynamic and replaced with actual values
* Text **outside** `{{ }}` is rendered literally
* These actions allow us to:

    * Pull in metadata
    * Render conditionals
    * Use functions and pipelines
    * Iterate over values

---

### ✅ Step-02-01: Valid Action Element

```yaml
# deployment.yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: {{ .Release.Name }}-{{ .Chart.Name }}
```

```bash
# Change to chart directory
cd helmbasics

# Render the manifest without applying it
helm template myapp101 .
```

> This will render a Deployment name like `myapp101-helmbasics`. The `helm template` command is **very useful during development and debugging**, as it shows the final Kubernetes manifest without actually applying it to a cluster.

---

### ❌ Step-02-02: Invalid Action Element

```yaml
# deployment.yaml
metadata:
  name: {{ something }}-{{ .Chart.Name }}
```

```bash
helm template myapp101 .
```

📌 **Error Output**:

```
Error: parse error at (helmbasics/templates/deployment.yaml:10): function "something" not defined
```

> ❗ This fails because `something` is **not defined in Helm’s built-in scope**. You can only use valid objects (like `.Chart`, `.Values`, etc.) or defined template functions.

---

## 🔤 Step-03: Template Function – `quote`

The `quote` function wraps a value in double quotes.

```yaml
annotations:
  app.kubernetes.io/managed-by: {{ quote .Release.Service }}
```

> This is helpful for ensuring your strings are properly quoted in YAML, especially when dealing with values that may contain spaces or special characters.

```bash
cd helmbasics
helm template myapp101 .
```

---

## 🔁 Step-04: Using Pipelines

Pipelines allow you to **chain multiple functions or actions together**, passing the output of one function as input to the next.

```yaml
annotations:
  # No pipeline
  app.kubernetes.io/managed-by: {{ quote .Release.Service }}
  
  # Same with pipeline
  app.kubernetes.io/managed-by: {{ .Release.Service | quote }}
```

> Pipelines improve readability and allow **layered transformations**. For example:
> `{{ .Release.Service | quote | lower }}`

---

## 🧩 Step-05: Functions – `default`, `lower`

Helm offers many built-in functions to transform or sanitize values.

### Example: `default` and `lower`

```yaml
# values.yaml
releaseName: "newrelease101"
replicaCount: 3
```

```yaml
annotations:
  app.kubernetes.io/name: {{ default "MYRELEASE101" .Values.releaseName | lower }}
spec:
  replicas: {{ default 1 .Values.replicaCount }}
```

> * `default`: Provides a fallback value if the user doesn’t supply one.
> * `lower`: Converts a string to lowercase.

This ensures your chart works **even if the user omits certain values** in their `values.yaml`.

```bash
helm template myapp101 .
```

---

## 🧼 Step-06: Controlling Whitespace with `{{- -}}`

Whitespace control is critical in YAML-sensitive environments like Kubernetes.

```yaml
annotations:
  leading-whitespace: "   {{- .Chart.Name }}    kalyan"
  trailing-whitespace: "   {{ .Chart.Name -}}    kalyan"
  leadtrail-whitespace: "   {{- .Chart.Name -}}    kalyan"
```

> * `{{-` trims whitespace **before** the statement
> * `-}}` trims whitespace **after** the statement
> * `{{- .Chart.Name -}}` trims **both**

```bash
helm template myapp101 .
```

---

## ✨ Step-07: `indent` and `nindent` Functions

These functions are useful when embedding **multi-line values** into YAML, such as `resources`, `configMaps`, or even `initContainers`.

### 🔹 Example:

```yaml
annotations:
  # Basic
  indenttest: "  {{- .Chart.Name | indent 4 -}}  "
  
  # With new line
  nindenttest: "  {{- .Chart.Name | nindent 4 -}}  "
```

> * `indent`: Adds indentation but **does not prepend a newline**
> * `nindent`: Adds both a newline and indentation

These are crucial for formatting nested blocks, especially inside `containers`, `volumes`, or config maps.

---

## 🧬 Step-08: Function – `toYaml`

The `toYaml` function converts a complex object like a dict, list, or nested structure into proper YAML formatting. It is often used with `nindent` to ensure alignment.

### 🔹 values.yaml

```yaml
resources: 
  limits:
    cpu: 100m
    memory: 128Mi
  requests:
    cpu: 100m
    memory: 128Mi
```

### 🔹 deployment.yaml

```yaml
spec:
  containers:
  - name: nginx
    image: ghcr.io/stacksimplify/kubenginx:4.0.0
    ports:
    - containerPort: 80
    resources:
    {{- toYaml .Values.resources | nindent 10 }}
```

> This renders the YAML block for `resources` with proper spacing and format.

### 🔍 Run Commands

```bash
helm template myapp101 .
helm install myapp101 . --dry-run
helm install myapp101 . --atomic
kubectl get pods
kubectl describe pod <POD-NAME>
helm uninstall myapp101
```

---

## ✅ Final Summary Table

| Concept           | Description                                    |                                           |
| ----------------- | ---------------------------------------------- | ----------------------------------------- |
| `{{ . }}`         | Root context object (represents current scope) |                                           |
| `quote`           | Wraps a string in double quotes                |                                           |
| \`                | \`                                             | Pipeline operator, chains transformations |
| `default`         | Provides a fallback if a value is not defined  |                                           |
| `lower` / `upper` | Converts string to lower or upper case         |                                           |
| `{{-`, `-}}`      | Trim leading or trailing whitespace            |                                           |
| `indent`          | Indents each line of the given block           |                                           |
| `nindent`         | Like `indent`, but adds a newline at the start |                                           |
| `toYaml`          | Converts an object to properly formatted YAML  |                                           |

---

## 📚 Additional Learning Resources

* 🔗 [Helm Functions Reference](https://helm.sh/docs/chart_template_guide/function_list/)
* 🔗 [Go Template Language Overview](https://pkg.go.dev/text/template)

