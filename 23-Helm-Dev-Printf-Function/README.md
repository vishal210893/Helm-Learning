# Helm Printf Function

## Step-01: Introduction

* The `printf` function in Helm templates is used to format strings similarly to how it works in Go or C-style languages.
* It allows you to dynamically construct strings by injecting values into placeholders, making it ideal for creating resource names, labels, or annotations that concatenate various variables.
* **Syntax:**

  ```gotpl
  {{ printf FORMAT_STRING ARG1 ARG2 ... }}
  ```
* Format specifiers:

    * `%s` – string
    * `%d` – integer
    * `%v` – default format for any value
* Reference: [Helm Docs - printf](https://helm.sh/docs/chart_template_guide/function_list/#printf)

---

## Step-02: Create a Named Template with `printf` Function

* File: `_helpers.tpl`
* Purpose: Generate a Kubernetes resource name by combining release and chart names with a hyphen

```gotpl
{{/* Kubernetes Resource Name: String Concat with Hyphen */}}
{{- define "helmbasics.resourceName" -}}
{{- printf "%s-%s" .Release.Name .Chart.Name -}}
{{- end }}
```

> This template uses `printf` to dynamically generate a string like `myapp1-helmbasics`

---

## Step-03: Call the Named Template in `deployment.yaml`

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: {{ include "helmbasics.resourceName" . }}-deployment 
  labels:
    app: nginx
```

> `include` is used because it returns a string, allowing chaining (e.g., adding `-deployment` suffix).

---

## Step-04: Test the Output

```bash
# Move to your chart directory
cd helmbasics

# Render the manifest
helm template myapp1 .

# Dry-run install to test rendered output
helm install myapp1 . --dry-run

# Actual install with resource creation
helm install myapp1 . --atomic

# Confirm the resource name
kubectl get deployments

# Uninstall release
helm uninstall myapp1
```

---

## Notes

* Always prefer `printf` when concatenating multiple strings or formatting values cleanly.
* This helps avoid manual spacing or fragile string construction in template logic.
* You can combine `printf` with `include` or `required` to enforce strict formatting and validation.
