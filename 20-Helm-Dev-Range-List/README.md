# Helm Development - Flow Control Range Action with List

## Step-01: Introduction

* The `range` action in Helm templates is a powerful way to iterate over lists (arrays), maps, or slices.
* It's commonly used to dynamically render repetitive Kubernetes manifests like multiple namespaces, deployments, or configMaps.
* This section demonstrates how to:

    * Loop over a list of simple values from `values.yaml`
    * Use the loop context with and without a Helm variable
    * Output multi-document YAML using `---` between each resource

---

## Step-02: Implement "Range Action" with "List of Values"

* **Use Case:** Loop over a simple list of namespaces and create one `Namespace` manifest for each.
* **Source File:** `backupfiles/namespace.yaml`
* **Target File:** `helmbasics/templates/namespace.yaml`

```yaml
# values.yaml
namespaces:
  - name: myapp1
  - name: myapp2
  - name: myapp3
```

```yaml
# helmbasics/templates/namespace.yaml
{{- range .Values.namespaces }}
apiVersion: v1
kind: Namespace
metadata:
  name: {{ .name }}
---
{{- end }}
```

### Test & Execution

```bash
cd helmbasics

# Render output for inspection
helm template myapp1 .

# Simulate the install
helm install myapp1 . --dry-run

# Perform actual install
helm install myapp1 . --atomic

# Confirm Helm release and created resources
helm list
helm status myapp1 --show-resources
kubectl get ns

# Uninstall to clean up
helm uninstall myapp1
```

**Important Notes:**

* The dot `.` inside the `range` block refers to the current item being iterated (e.g., `myapp1`, `myapp2`).
* The `---` between each block ensures proper YAML document separation in a single file.

---

## Step-03: Implement "Range Action" with "List of Values" and Helm Variables

* **Use Case:** Define a named variable (`$environment`) to hold the current loop item for better readability and nesting flexibility.
* **Source File:** `backupfiles/namespace-with-variable.yaml`
* **Target File:** `helmbasics/templates/namespace-with-variable.yaml`

```yaml
# values.yaml
environments:
  - name: dev
  - name: qa
  - name: uat  
  - name: prod
```

```yaml
# helmbasics/templates/namespace-with-variable.yaml
{{- range $environment := .Values.environments }}
apiVersion: v1
kind: Namespace
metadata:
  name: {{ $environment.name }}
---
{{- end }}
```

### Test & Execution

```bash
cd helmbasics

# Render for dry-run verification
helm template myapp1 .

# Dry-run install
helm install myapp1 . --dry-run

# Actual install with rollback safety
helm install myapp1 . --atomic

# Check release and resources
helm list
helm status myapp1 --show-resources
kubectl get ns

# Cleanup
helm uninstall myapp1
```

**Additional Tips:**

* Using a named variable (`$environment`) helps when:

    * You want to reference `.Values` from the parent scope alongside the loop variable
    * You're writing deeply nested templates or passing sub-values to included templates
* Always ensure `---` is used when looping through multiple Kubernetes manifests in one template file to maintain valid multi-document output.
