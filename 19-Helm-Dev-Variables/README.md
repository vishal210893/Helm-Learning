# Helm Development – Using Variables in Templates

---

## 🧭 Step-01: Introduction

In Helm templates, variables are a powerful mechanism to simplify repetitive expressions, store intermediate results (e.g., transformations using `quote`, `upper`, `toYaml`), and improve template readability and maintainability.

Variables in Helm are defined using the Go templating engine’s `:=` operator within `{{ }}` blocks. Once declared, they can be referenced just like any other object using the `$` prefix.

---

## 🧪 Step-02: Define and Use Variables in Helm Templates

Let’s see a practical case where we declare a variable and use it in a resource annotation.

### 🛠️ deployment.yaml

```yaml
# Declare a variable at the beginning of the template file
{{- $chartname := .Chart.Name -}}

apiVersion: apps/v1
kind: Deployment
metadata:
  name: {{ .Release.Name }}-{{ .Chart.Name }}
  labels:
    app: nginx
spec:
  replicas: 1
  selector:
    matchLabels:
      app: nginx
  template:
    metadata:
      labels:
        app: nginx
      {{- with .Values.podAnnotations }}
      annotations:
        {{- toYaml . | nindent 8 }}
        appManagedBy: {{ $.Release.Service }}
        appHelmChart: {{ $chartname }}
      {{- end }}
    spec:
      containers:
      - name: nginx
        image: ghcr.io/stacksimplify/kubenginx:4.0.0
        ports:
        - containerPort: 80
```

### 📂 Change to Chart Directory and Render Template

```bash
cd helmbasics  

# Render Helm template
helm template myapp101 .

# Perform a dry-run install
helm install myapp101 . --dry-run
```

### 🔍 Observation

* The annotation field `appHelmChart` should be populated with the value of `.Chart.Name`.
* You’ll also see the `appManagedBy` using the root-level value from `.Release.Service` thanks to the `with` block + `$` usage.

---

## 🧪 Step-03: Combine Variables with Pipeline Functions

We can take this further by modifying the variable through functions (e.g., `quote`, `upper`) and using that enhanced value downstream.

### 🔁 Example

```yaml
{{- $chartname := .Chart.Name | quote | upper -}}

apiVersion: apps/v1
kind: Deployment
metadata:
  name: {{ .Release.Name }}-{{ .Chart.Name }}
  labels:
    app: nginx
spec:
  replicas: 1
  selector:
    matchLabels:
      app: nginx
  template:
    metadata:
      labels:
        app: nginx
      {{- with .Values.podAnnotations }}
      annotations:
        {{- toYaml . | nindent 8 }}
        appManagedBy: {{ $.Release.Service }}
        appHelmChart: {{ $chartname }}
      {{- end }}
    spec:
      containers:
      - name: nginx
        image: ghcr.io/stacksimplify/kubenginx:4.0.0
        ports:
        - containerPort: 80
```

### 🧪 Test & Verify

```bash
cd helmbasics

# Preview rendered manifest
helm template myapp101 .

# Dry-run to validate full installation behavior
helm install myapp101 . --dry-run

# Real install
helm install myapp101 . --atomic

# Check installed releases
helm list

# Check running pods
kubectl get pods

# Inspect manifest
helm get manifest myapp101

# Cleanup
helm uninstall myapp101
```

---

## 📌 Key Takeaways

* Use variables to **store computed expressions** or reused values.
* Variables improve readability and performance (values are computed once).
* Always prefix variables with `$` (e.g., `$chartname`) to distinguish them.
* Combine with pipelines to apply transformations on-the-fly (`| quote`, `| upper`, `| lower`, etc.).
* Be mindful of scope — define variables at the appropriate level (file-wide or within a block).
