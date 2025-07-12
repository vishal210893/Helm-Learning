---

# 🧠 Helm Built-in Objects

> Understand and experiment with Helm's built-in template objects like `.Release`, `.Chart`, `.Values`, and `.Files`.

---

## 📘 Step-01: Introduction

Helm templates are powered by the Go template engine. It injects **built-in objects** into your templates which represent different aspects of:

* The release being installed (`.Release`)
* The chart metadata (`.Chart`)
* The values passed (`.Values`)
* The cluster environment (`.Capabilities`)
* The current template being rendered (`.Template`)
* Files bundled in the chart (`.Files`)

### ✨ Built-in Objects:

* `.Release`
* `.Chart`
* `.Values`
* `.Capabilities`
* `.Template`
* `.Files`

---

## 🛠️ Step-02: Create a Simple Helm Chart and Prepare for Testing

```bash
# Create a sample Helm chart
helm create builtinobjects

# Clean up the auto-generated NOTES.txt content
cd builtinobjects/templates
> NOTES.txt

# Return to chart root
cd ..

# Perform a dry-run install
helm install myapp1 . --dry-run
```

---

## 🔍 Step-03: Object: Root (Dot or Period `.`)

The dot (`.`) refers to the **root context** object.

📄 Add this to `templates/NOTES.txt`:

```gotemplate
{{/* Root or Dot or Period Object */}}
Root Object: {{ . }}
```

```bash
# Dry-run to inspect the output
helm install myapp101 . --dry-run
```

---

## 📦 Step-04: Object: `.Release`

This object contains metadata about the Helm release:

📄 Update `NOTES.txt`:

```gotemplate
{{/* Release Object */}}
Release Name: {{ .Release.Name }}
Release Namespace: {{ .Release.Namespace }}
Release IsUpgrade: {{ .Release.IsUpgrade }}
Release IsInstall: {{ .Release.IsInstall }}
Release Revision: {{ .Release.Revision }}
Release Service: {{ .Release.Service }}
```

```bash
cd builtinobjects
helm install myapp101 . --dry-run
```

✅ Sample Output:

```
Release Name: myapp101
Release Namespace: default
Release IsUpgrade: false
Release IsInstall: true
Release Revision: 1
Release Service: Helm
```

---

## 📘 Step-05: Object: `.Chart`

This represents the metadata from `Chart.yaml`.

📄 Add to `NOTES.txt`:

```gotemplate
{{/* Chart Object */}}
Chart Name: {{ .Chart.Name }}
Chart Version: {{ .Chart.Version }}
Chart AppVersion: {{ .Chart.AppVersion }}
Chart Type: {{ .Chart.Type }}
Chart Name and Version: {{ .Chart.Name }}-{{ .Chart.Version }}
```

```bash
helm install myapp101 . --dry-run
```

✅ Sample Output:

```
Chart Name: builtinobjects
Chart Version: 0.1.0
Chart AppVersion: 0.1.0
Chart Type: application
Chart Name and Version: builtinobjects-0.1.0
```

🔗 [Chart.yaml Reference](https://helm.sh/docs/topics/charts/#the-chartyaml-file)

---

## ⚙️ Step-06: Objects: `.Values`, `.Capabilities`, `.Template`

📄 Add to `NOTES.txt`:

```gotemplate
{{/* Values Object */}}
Replica Count: {{ .Values.replicaCount }}
Image Repository: {{ .Values.image.repository }}
Service Type: {{ .Values.service.type }}

{{/* Capabilities Object */}}
Kubernetes Cluster Version Major: {{ .Capabilities.KubeVersion.Major }}
Kubernetes Cluster Version Minor: {{ .Capabilities.KubeVersion.Minor }}
Kubernetes Cluster Version: {{ .Capabilities.KubeVersion }} and {{ .Capabilities.KubeVersion.Version }}
Helm Version: {{ .Capabilities.HelmVersion }}
Helm Version Semver: {{ .Capabilities.HelmVersion.Version }}

{{/* Template Object */}}
Template Name: {{ .Template.Name }}
Template Base Path: {{ .Template.BasePath }}
```

```bash
helm install myapp101 . --dry-run
```

✅ Sample Output:

```
Replica Count: 1
Image Repository: ghcr.io/stacksimplify/kubenginxhelm
Service Type: NodePort

Kubernetes Cluster Version Major: 1
Kubernetes Cluster Version Minor: 27
Kubernetes Cluster Version: v1.27.2 and v1.27.2
Helm Version: {v3.12.1 ...}
Helm Version Semver: v3.12.1

Template Name: builtinobjects/templates/NOTES.txt
Template Base Path: builtinobjects/templates
```

---

## 📂 Step-07: Object: `.Files`

Helm’s `.Files` object lets you read and use static files bundled in your chart.

📄 Add to `NOTES.txt`:

```gotemplate
{{/* Files Object */}}
File Get: {{ .Files.Get "myconfig1.toml" }}
File Glob as Config: {{ (.Files.Glob "config-files/*").AsConfig }}
File Glob as Secret: {{ (.Files.Glob "config-files/*").AsSecrets }}
File Lines: {{ .Files.Lines "myconfig1.toml" }}
File Lines: {{ .Files.Lines "config-files/myconfig2.toml" }}
File Glob: {{ .Files.Glob "config-files/*" }}
```

```bash
cd builtinobjects
helm install myapp101 . --dry-run
```

✅ Sample Output:

```toml
File Get: message1 = Hello from config 1 line1
message2 = Hello from config 1 line2
message3 = Hello from config 1 line3

File Glob as Config:
myconfig2.toml: |-
  appName: myapp2
  appType: db
  appConfigEnable: true
myconfig3.toml: |-
  appName: myapp3
  appType: app
  appConfigEnable: false

File Glob as Secret:
myconfig2.toml: <base64>
myconfig3.toml: <base64>

File Lines: [message1 = Hello from config 1 line1 ...]
```

🔗 [Accessing Files in Templates – Helm Docs](https://helm.sh/docs/chart_template_guide/accessing_files/)

---

## 📚 Additional References

* 🧩 [Helm Built-In Objects](https://helm.sh/docs/chart_template_guide/builtin_objects/)
* 📄 [Helm Chart.yaml Fields](https://helm.sh/docs/topics/charts/#the-chartyaml-file)

---