# Create and Package Helm Charts

## Step-01: Introduction

This guide covers creating and packaging Helm Charts from scratch with practical steps and versioning. We'll go through:

* Creating a Helm chart using `helm create`
* Customizing the chart with your own Docker image
* Modifying the service to use NodePort
* Updating metadata such as `version` and `appVersion`
* Installing and testing the chart
* Packaging it for distribution or upload to a repository
* Using the `helm show` command to inspect packaged charts

> **Docker Image Used:**
> [ghcr.io/stacksimplify/kubenginx](https://github.com/users/stacksimplify/packages/container/package/kubenginx)

---

## Step-02: Helm Create Chart

```bash
helm create <CHART-NAME>
helm create myfirstchart
```

**Observation:**

* Generates a default starter Helm chart directory.
* Useful for learning structure and quickly bootstrapping custom charts.

---

## Step-03: Update `values.yaml` with Docker Image

```yaml
image:
  repository: ghcr.io/stacksimplify/kubenginx
  pullPolicy: IfNotPresent
  tag: ""
```

Then verify in `templates/deployment.yaml` that the deployment `.spec.template.spec.containers.image` refers to these values:

```gotpl
image: "{{ .Values.image.repository }}:{{ .Values.image.tag | default .Chart.AppVersion }}"
```

---

## Step-04: Modify Service Type to NodePort

Update `values.yaml`:

```yaml
service:
  type: NodePort
  port: 80
  nodePort: 31231
```

Update `templates/service.yaml`:

```yaml
spec:
  type: {{ .Values.service.type }}
  ports:
    - port: {{ .Values.service.port }}
      nodePort: {{ .Values.service.nodePort }}
```

---

## Step-05: Update `Chart.yaml`

```yaml
version: 1.0.0
description: A Helm Chart with NodePort Service
appVersion: "1.0.0"
```

---

## Step-06: Install and Test Chart (v1.0.0)

```bash
cd myfirstchart
helm install myapp1v1 .

helm list
helm status myapp1v1 --show-resources

kubectl get pods
kubectl get svc

# Access in browser
http://localhost:31231
```

---

## Step-07: Helm Package v1.0.0

```bash
helm package myfirstchart/ --destination packages/
```

**Expected Output:**

```bash
packages/myfirstchart-1.0.0.tgz
```

---

## Step-08: Update to v2.0.0 and Package Again

Update `Chart.yaml`:

```yaml
version: 2.0.0
description: A Helm Chart with NodePort Service
appVersion: "2.0.0"
```

Package again:

```bash
helm package myfirstchart/ -d packages/
```

**Now you should have:**

```bash
packages/myfirstchart-1.0.0.tgz
packages/myfirstchart-2.0.0.tgz
```

---

## Step-09: Install Packaged Chart by Path (v2.0.0)

```bash
helm install myapp1v2 packages/myfirstchart-2.0.0.tgz --set service.nodePort=31232

kubectl get pods
kubectl get svc

helm list
helm status myapp1v2 --show-resources

# Access in browser
http://localhost:31232
```

---

## Step-10: Helm Package with Custom Version and App Version (v3.0.0)

```bash
helm package myfirstchart/ --app-version "3.0.0" --version "3.0.0" --destination packages/
```

---

## Step-11: Install and Test Packaged Chart v3.0.0

```bash
helm install myapp1v3 packages/myfirstchart-3.0.0.tgz --set service.nodePort=31233

kubectl get pods
kubectl get svc

helm list
helm status myapp1v3 --show-resources

# Access in browser
http://localhost:31233
```

---

## Step-12: Uninstall All Helm Releases

```bash
helm list
helm uninstall myapp1v1
helm uninstall myapp1v2
helm uninstall myapp1v3
```

---

## Step-13: Helm Show Commands

These commands are useful for inspecting charts:

```bash
helm show chart myfirstchart/
helm show chart packages/myfirstchart-2.0.0.tgz

helm show values myfirstchart/
helm show values packages/myfirstchart-2.0.0.tgz

helm show readme myfirstchart/

helm show all myfirstchart/
helm show all packages/myfirstchart-2.0.0.tgz
```

---

> Tip: The `packages/` folder can now be version-controlled or uploaded to an internal Helm repository (e.g., Nexus, ChartMuseum, GitHub Pages). You can use `helm repo index` to build an index.yaml from your packaged charts.
