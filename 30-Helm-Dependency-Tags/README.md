# Helm Dependency - Using Tags

## Step-01: Introduction

* Helm provides the ability to group and control subchart inclusion using `tags` instead of `condition`.
* This is especially helpful when managing multiple dependencies that logically belong to the same group, such as `frontend`, `backend`, `database`, etc.
* Tags allow you to enable or disable entire sets of dependencies in a clean, scalable way.

---

## Step-02: Define Tags in `Chart.yaml`

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
    repository: "https://stacksimplify.github.io/helm-charts/"
    alias: childchart4dev
    tags:
      - frontend 

  - name: mychart4
    version: "0.1.0"
    repository: "https://stacksimplify.github.io/helm-charts/"
    alias: childchart4qa1
    tags:
      - frontend   

  - name: mychart4
    version: "0.1.0"
    repository: "https://stacksimplify.github.io/helm-charts/"
    alias: childchart4qa2
    tags:
      - frontend        

  - name: mychart2
    version: "0.4.0"
    repository: "https://stacksimplify.github.io/helm-charts/"
    alias: childchart2
    tags:
      - backend
```

### Notes:

* Every alias is associated with one or more tags.
* This enables or disables all dependencies matching a tag via the `values.yaml` file or CLI overrides.

---

## Step-03: Use Case 1 - Both `frontend` and `backend` are `false`

```yaml
# values.yaml
tags:
  frontend: false
  backend: false
```

```bash
# Install Helm chart
helm install myapp1 parentchart/ --atomic

# List pods
kubectl get pods
```

### Observation:

* Only the parent chart’s pod will be created.
* All tagged subcharts will be skipped.

---

## Step-04: Use Case 2 - `backend: true`, `frontend: false`

```bash
# Upgrade release with backend enabled
helm upgrade myapp1 parentchart/ --atomic --set tags.backend=true

# List pods
kubectl get pods
```

### Observation:

* Two pods should be running:

    * One from the parent chart
    * One from `childchart2` (backend tag)

---

## Step-05: Use Case 3 - `backend: true`, `frontend: true`

```bash
# Upgrade release with both backend and frontend enabled
helm upgrade myapp1 parentchart/ --atomic \
  --set tags.backend=true \
  --set tags.frontend=true

# List pods
kubectl get pods
```

### Observation:

* Five pods should be running:

    * One from the parent chart
    * One from `childchart2` (backend)
    * Three from frontend-tagged subcharts: `childchart4dev`, `childchart4qa1`, `childchart4qa2`

---

## Step-06: Clean Up

```bash
helm uninstall myapp1
```

---

## Additional Notes:

* You can also define default tag values directly in the parent chart’s `values.yaml`:

```yaml
tags:
  frontend: true
  backend: false
```

* To selectively enable a tag only for certain environments (e.g., dev, qa, prod), use Helm value files like `values-dev.yaml`, `values-qa.yaml` with corresponding tag configurations.
