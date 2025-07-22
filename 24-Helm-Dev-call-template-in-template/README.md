# Helm Named Templates - Call Template in Template

## Step-01: Introduction

* Helm allows one named template to invoke another named template, facilitating better reuse and modularity in complex chart logic.
* This approach supports a clean separation of concerns where small building blocks (named templates) can be combined for structured output.
* Useful for dynamically generating content like labels, annotations, resource names, etc., without duplicating logic.

---

## Step-02: Update `_helpers.tpl`

* Goal: Update the existing `helmbasics.labels` named template to include the output of another named template `helmbasics.resourceName`.
* This demonstrates a **template calling another template**, also known as *template-in-template*.

```gotpl
{{/* Named Template: Common Labels */}}
{{- define "helmbasics.labels" -}}
    app.kubernetes.io/managed-by: helm
    app: nginx
    chartname: {{ .Chart.Name }}
    template-in-template: {{ include "helmbasics.resourceName" . }}
{{- end }}

{{/* Named Template: Kubernetes Resource Name */}}
{{- define "helmbasics.resourceName" -}}
{{- printf "%s-%s" .Release.Name .Chart.Name -}}
{{- end }}
```

* `include "helmbasics.resourceName" .` is used to pass the current context (`.`) and fetch the result as a string.
* This output is then injected as the value for the `template-in-template` label.
* `printf` is used inside `resourceName` to format and concatenate release and chart names.

---

## Step-03: Usage in a Kubernetes Manifest (e.g., `deployment.yaml`)

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: {{ include "helmbasics.resourceName" . }}-deployment
  labels:
{{ include "helmbasics.labels" . | indent 4 }}
```

> Using `include` makes it possible to return and process string-based templates such as label maps, unlike `template` which is only for output rendering.

---

## Step-04: Test the Changes

```bash
# Navigate to the chart directory
cd helmbasics

# Render templates locally
helm template myapp1 .

# Perform a dry-run install to validate rendered manifests
helm install myapp1 . --dry-run

# Actual install (creates Kubernetes resources)
helm install myapp1 . --atomic

# Verify output
kubectl get deployments
kubectl get deployment myapp1-helmbasics-deployment -o yaml | grep template-in-template

# Cleanup
helm uninstall myapp1
```

---

## Notes

* `include` is preferred for named templates that return strings and when the output will be part of another template string.
* You can nest templates as deeply as needed, provided each includes its required scope (`.`).
* Moving logic to named templates improves maintainability, especially when label or name logic must be reused across multiple manifests.
