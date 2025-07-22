# Helm Dependency - Condition with Alias

## Step-01: Introduction

* Helm allows defining multiple instances of the same subchart using different aliases.
* The `condition` field enables fine-grained control over which aliased subcharts are rendered and installed.
* Values for controlling these conditions must use the alias names, not the original subchart name.

---

## Step-02: Configure `Chart.yaml` with Aliases and Conditions

```yaml
# parentchart/Chart.yaml
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
    condition: childchart4dev.enabled

  - name: mychart4
    version: "0.1.0"
    repository: "https://stacksimplify.github.io/helm-charts/"
    alias: childchart4qa
    condition: childchart4qa.enabled  

  - name: mychart2
    version: "0.4.0"
    repository: "https://stacksimplify.github.io/helm-charts/"
    alias: childchart2
    condition: childchart2.enabled
```

### Notes:

* Each alias must define its own `condition` referencing the alias name.
* If `condition` is not explicitly disabled, Helm assumes the chart is enabled by default.

---

## Step-03: Update `values.yaml` to Control Aliased Subcharts

```yaml
# parentchart/values.yaml
childchart4dev:
  enabled: false

childchart4qa:
  enabled: true

childchart2:
  enabled: false
```

### Notes:

* Setting `enabled: false` disables rendering and installation of that aliased subchart.
* Use the alias name exactly in the `values.yaml` structure.

---

## Step-04: Deploy and Validate Results

```bash
# Download dependencies
helm dependency update parentchart/

# Install parent chart
helm install myapp1 parentchart/ --atomic

# Validate installation
helm list
helm status myapp1 --show-resources
kubectl get deploy
kubectl get pods
kubectl get svc
```

### Expected Outcome:

* Only the parent chart and `childchart4qa` (enabled alias) will have Kubernetes resources deployed.
* `childchart4dev` and `childchart2` will be skipped entirely due to `enabled: false`.

---

## Step-05: Uninstall Release

```bash
helm uninstall myapp1
```

---

## Additional Tip:

To override values without modifying `values.yaml`, you can use CLI:

```bash
helm install myapp1 parentchart/ \
  --set childchart4dev.enabled=false \
  --set childchart4qa.enabled=true \
  --set childchart2.enabled=false
```

This is particularly helpful in automation pipelines and dynamic deployment environments.
