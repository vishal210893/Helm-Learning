# Helm Dependency - Condition

## Step-01: Introduction

* `condition` is used to enable or disable the installation of subcharts (child charts) from the parent chart.
* You can control whether a subchart should be rendered and installed by toggling a boolean flag in the parent chart’s `values.yaml`.
* This is particularly useful for optional dependencies or environments where certain components may be excluded.

---

## Step-02: Configure `Chart.yaml` with `condition`

```yaml
# parentchart/Chart.yaml
dependencies:
  - name: mychart4
    version: "0.1.0"
    repository: "https://stacksimplify.github.io/helm-charts/"
    alias: childchart4
    condition: mychart4.enabled
  - name: mychart2
    version: "0.4.0"
    repository: "https://stacksimplify.github.io/helm-charts/"
    alias: childchart2
    condition: mychart2.enabled
```

### Notes:

* `condition` takes a path from `.Values`.
* Even if the condition is declared, Helm enables the subchart by default unless explicitly disabled.
* Aliases (like `childchart4`) are the chart instances used in rendering, but the `condition` is still keyed off the **original name** (`mychart4`).

---

## Step-03: Deploy and Test – Default Behavior (Both Enabled)

```bash
# Download the dependencies
helm dependency update parentchart/

# Install parent chart
helm install myapp1 parentchart/ --atomic

# Verify installation
helm list
helm status myapp1 --show-resources
kubectl get deploy
kubectl get pods
kubectl get svc
```

### Access Expected Services:

* Parent chart service: `http://localhost:<port>`
* `mychart4` (via `childchart4` alias): `http://localhost:<port>`
* `mychart2` (via `childchart2` alias): `http://localhost:31232`

---

## Step-04: Disable Child Charts in `values.yaml`

```yaml
# parentchart/values.yaml
mychart4:
  enabled: false

mychart2:
  enabled: false
```

---

## Step-05: Deploy and Test – When Subcharts Are Disabled

```bash
# Update dependencies again
helm dependency update parentchart/

# Install chart
helm install myapp1 parentchart/ --atomic

# Verify results
helm list
helm status myapp1 --show-resources
kubectl get deploy
kubectl get pods
kubectl get svc
```

### Observation:

* Only parent chart resources should be created.
* `childchart4` and `childchart2` will be skipped entirely from the rendered manifests.

---

## Step-06: Cleanup

```bash
helm uninstall myapp1
```

---

## Extra Note:

To override the condition during install time without editing `values.yaml`, use:

```bash
helm install myapp1 parentchart/ --set mychart4.enabled=false --set mychart2.enabled=false
```

This makes the chart more flexible in CI/CD pipelines and staging/production workflows.
