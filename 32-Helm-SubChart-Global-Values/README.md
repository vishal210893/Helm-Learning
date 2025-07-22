# Helm Sub Charts - Use Global Values in Sub Charts

## Step-01: Introduction

* In Helm, **global values** provide a mechanism for sharing configuration values between a parent chart and its subcharts.
* This is especially useful when you want to enforce common settings (e.g., `replicaCount`, `image.tag`, `namespace`, etc.) across multiple charts without repeating them individually in each subchart.
* This walkthrough demonstrates how to define and use global values in subcharts, while managing dependencies manually (via `file://`).

---

## Step-02: Review Chart.yaml

```yaml
apiVersion: v2
name: parentchart
description: Learn Helm Dependency Concepts
type: application
version: 0.1.0
appVersion: "1.16.0"
dependencies:
  - name: mychart4
    version: "0.1.0"
    repository: "file://charts/mychart4"
    alias: childchart4
    tags:
      - frontend
  - name: mychart2
    version: "0.4.0"
    repository: "file://charts/mychart2"
    alias: childchart2
    tags:
      - backend
```

> The dependencies are referenced via local file paths to allow editing subcharts directly. The use of tags allows grouping subcharts logically.

---

## Step-03: Pull Charts into Local Directory

```bash
# Navigate to the parentchart/charts directory
cd parentchart/charts

# Pull and untar mychart4 from remote repository
helm pull https://stacksimplify.github.io/helm-charts/mychart4-0.1.0.tgz --untar

# Pull and untar mychart2 from remote repository
helm pull https://stacksimplify.github.io/helm-charts/mychart2-0.4.0.tgz --untar

# Clean up compressed files
rm -rf *.tgz
```

> This gives editable access to subchart sources under `parentchart/charts/`.

---

## Step-04: Build Dependencies

```bash
# From the parentchart directory
cd ..

# List dependencies
helm dependency list

# Sample output:
# NAME    	VERSION	REPOSITORY             	STATUS
# mychart4	0.1.0  	file://charts/mychart4 	unpacked
# mychart2	0.4.0  	file://charts/mychart2	unpacked

# Run dependency update to generate Chart.lock and verify
helm dependency update

# Validate charts directory
ls charts/

# Cleanup the generated .tgz files (optional)
rm charts/*.tgz
```

---

## Step-05: Define Global Values in Parent Chart

**File: `parentchart/values.yaml`**

```yaml
# Define Global Values
global:
  replicaCount: 4
```

> This value will be available in subcharts under `.Values.global`.

---

## Step-06: Update Deployment Templates to Use Global Value

### 1. **Parent Chart Deployment**

**File: `parentchart/templates/deployment.yaml`**

```yaml
replicas: {{ .Values.global.replicaCount }}
```

### 2. **Subchart mychart4 Deployment**

**File: `charts/mychart4/templates/deployment.yaml`**

```yaml
replicas: {{ .Values.global.replicaCount }}
```

### 3. **Subchart mychart2 Deployment**

**File: `charts/mychart2/templates/deployment.yaml`**

```yaml
replicas: {{ .Values.global.replicaCount }}
```

> Global values are automatically merged into `.Values.global` in subcharts and are accessible without needing alias-specific names.

---

## Step-07: Deploy and Verify

```bash
# Ensure you're in parentchart directory
cd parentchart

# Install the chart
helm install myapp1 . --atomic

# Validate that all charts used the global value
kubectl get pods
```

### Expected Output:

* 1 Deployment (possibly) for parentchart
* 1 Deployment each for `childchart4` and `childchart2`
* Each Deployment should have 4 pods (replica count = 4)

---

## Step-08: Clean Up

```bash
# Uninstall the release
helm uninstall myapp1
```

---

## Additional Notes:

* `global` is a **reserved key** in Helm. It’s automatically merged and passed to all templates, including subcharts.
* Use it for:

    * Common values (e.g., image versions, replicaCount, app labels)
    * Managing environment-level configurations in complex Helm stacks
* If both the subchart and global define the same value, Helm does **not override** the subchart-specific value unless it’s not set in that chart. Global values are only a fallback.
