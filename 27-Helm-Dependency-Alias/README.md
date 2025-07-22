# Helm Dependency - Alias

## Step-01: Introduction

This section focuses on advanced dependency handling in Helm using:

* `alias`: to reuse the same subchart multiple times under different names
* overriding subchart values from the parent chart
* running and validating deployments of aliased subcharts independently

---

## Step-02: Update Parent Chart's Service Configuration

The parent chart defines a `NodePort` service for demonstration.

```yaml
# parentchart/values.yaml
service:
  type: NodePort
  port: 80
```

---

## Step-03: Use `alias` in `Chart.yaml` to Reuse Subcharts

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
  - name: mychart4
    version: "0.1.0"
    repository: "https://stacksimplify.github.io/helm-charts/"
    alias: childchart4qa
  - name: mychart2
    version: "0.4.0"
    repository: "https://stacksimplify.github.io/helm-charts/"
    alias: childchart2
```

### Why alias?

* You can use the same chart (`mychart4`) multiple times with different configurations.
* Helm treats each alias as a separate chart instance.

---

## Step-04: Deploy and Test the Helm Chart

### Update Dependencies

```bash
helm dependency update parentchart/
```

### Install the Parent Chart

```bash
helm install myapp1 parentchart/ --atomic
```

### Verify Install

```bash
helm list
helm status myapp1 --show-resources
kubectl get deploy
kubectl get pods
kubectl get svc
```

### Access Services

Use the ports from the `kubectl get svc` output to access:

* `parentchart`: `http://localhost:<parent-nodeport>`
* `childchart4dev`: `http://localhost:<dev-nodeport>`
* `childchart4qa`: `http://localhost:<qa-nodeport>`
* `childchart2` (from `mychart2`): `http://localhost:31232` (if `nodePort` is set in values)

---

## Step-05: Uninstall Helm Release

```bash
helm uninstall myapp1
```

---

## Additional Notes

* If you need to override values for a specific alias, use nested values in `parentchart/values.yaml` like:

```yaml
childchart4dev:
  service:
    nodePort: 31230
childchart4qa:
  service:
    nodePort: 31231
childchart2:
  service:
    nodePort: 31232
```

This ensures each aliased subchart receives unique configuration.
