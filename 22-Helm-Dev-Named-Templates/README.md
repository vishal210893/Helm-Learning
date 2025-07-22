# Helm Development - Named Templates

## Step-01: Introduction

* Named templates are reusable blocks of template logic that can be defined once and reused across multiple Helm template files.
* They’re declared using `{{ define "template.name" }}` and called using either `{{ template "template.name" . }}` or `{{ include "template.name" . }}`.
* The difference:

    * `template` is an action and cannot be piped into other functions
    * `include` is a function and returns a string, so it can be piped and nested

---

## Step-02: Create a Named Template

* File: `deployment.yaml`
* Define a named template for common labels

```yaml
{{/* Common Labels */}}
{{- define "helmbasics.labels" }}
    app: nginx
{{- end }}
```

---

## Step-03: Call the Named Template Without Scope

* File: `deployment.yaml`

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: {{ .Release.Name }}-{{ .Chart.Name }}-deployment 
  labels:
  {{- template "helmbasics.labels" }}
```

> Since scope `.` is not passed, the template has no access to root objects like `.Chart.Name`.

---

## Step-04: Test Without Scope

```bash
cd helmbasics
helm template myapp101 .
helm install myapp101 . --dry-run
```

**Expected:**

* Only static values will render
* Any dynamic reference (e.g., `.Chart.Name`) will be empty or throw an error if used in the template

---

## Step-05: Add Builtin Object `.Chart.Name` to Template

```yaml
{{/* Common Labels */}}
{{- define "helmbasics.labels" }}
    app: nginx
    chartname: {{ .Chart.Name }}
{{- end }}
```

> This now requires `.Chart.Name`, so the template must receive the scope via `.`

---

## Step-06: Fix Scope Passing by Adding Dot

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: {{ .Release.Name }}-{{ .Chart.Name }}-deployment 
  labels:
  {{- template "helmbasics.labels" . }}
```

> The `.` at the end passes the current scope into the named template, making built-in objects accessible inside it.

---

## Step-07: Test With Scope

```bash
helm template myapp101 .
helm install myapp101 . --dry-run
```

**Expected:**

* `chartname` label should correctly render with the actual chart name

---

## Step-08: Test With Function Piping (Fails)

```yaml
labels:
  {{- template "helmbasics.labels" . | upper }}
```

```bash
helm template myapp101 .
```

**Observation:**

* This fails because `template` is an action, not a function.
* Actions cannot return values to be piped.

---

## Step-09: Fix Using `include` Function

```yaml
labels:
  {{- include "helmbasics.labels" . | upper }}
```

**Observation:**

* `include` returns a string, so it can be piped into `upper`, `trim`, `indent`, etc.
* This works and results in all keys/values printed in upper case.

---

## Step-10: Move to `_helpers.tpl`

* Move the `define` block to `_helpers.tpl` for better organization

```gotpl
{{/* Common Labels */}}
{{- define "helmbasics.labels" }}
    app: nginx
    chartname: {{ .Chart.Name }}
{{- end }}
```

> Files beginning with `_` are helper-only files. They are not rendered into manifests but can be referenced anywhere in templates.

---

## Step-11: Final Testing

```bash
helm template myapp101 .
helm install myapp101 . --dry-run
```

**Expected:**

* No error
* `labels` render correctly with the `.Chart.Name` and `app: nginx`
* If `upper` is applied, output should be in uppercase

---

## Notes:

* Always prefer storing reusable snippets in `_helpers.tpl` for maintainability.
* Use `template` when embedding reusable blocks directly in templates without chaining.
* Use `include` when you need to manipulate the output, e.g., indenting or converting case.
* You can use `required` or `default` inside named templates for more robust error handling or fallbacks.
