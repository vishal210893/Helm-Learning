# Helm Dependency - Override Subchart Values

## Step-01: Introduction

* In Helm, parent charts can override values defined in subcharts using the same key structure as the subchart’s `values.yaml`.
* This is especially useful when you want to control replica counts, image tags, resource requests, or any other configuration centrally without modifying the original subchart.

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
    repository: "https://stacksimplify.github.io/helm-charts/"
    condition: mychart4.enabled
  - name: mychart2
    version: "0.4.0"
    repository: "https://stacksimplify.github.io/helm-charts/"
    condition: mychart2.enabled
```

> Both dependencies are gated by `enabled` conditions, allowing selective control.

---

## Step-03: Inspect Default Values in Subcharts

```bash
# Move to working directory
cd 31-Helm-Dependency-Override-Subchart-Values

# View default values for mychart4
helm show values parentchart/charts/mychart4-0.1.0.tgz

# View default values for mychart2
helm show values parentchart/charts/mychart2-0.4.0.tgz
```

> Check for keys like `replicaCount`, `image`, `service`, etc., which are commonly overridden.

---

## Step-04: Override Subchart Values in Parent Chart’s values.yaml

```yaml
# parentchart/values.yaml
mychart4:
  enabled: true
  replicaCount: 3

mychart2:
  enabled: true
  replicaCount: 3
```

> These override values will be passed to the subchart during rendering, replacing the defaults.

---

## Step-05: Deploy and Verify

```bash
# Update dependencies (in case they were modified)
helm dependency update parentchart/

# Install the parent chart
helm install myapp1 parentchart/ --atomic

# Verify resources
helm list
helm status myapp1 --show-resources

# Inspect deployments and pods
kubectl get deploy
kubectl get pods
```

### Expected Outcome:

* You should see:

    * **3 pods** for `mychart4`
    * **3 pods** for `mychart2`
    * **1 pod** for `parentchart` (if it has a workload)
* This confirms the `replicaCount` values were successfully overridden.

---

## Step-06: Access Services

```bash
kubectl get svc
```

> Use the NodePort or ClusterIP values returned to access applications like:

```
http://localhost:<port>
```

---

## Step-07: Clean Up

```bash
helm uninstall myapp1
```

> This will remove all workloads created by the parent and subcharts.

---

## Additional Notes:

* To override deeper values (e.g., nested maps), match the exact hierarchy from the subchart’s `values.yaml`.
* You can also use `--set` or `--values` flags to override inline or with additional files:

  ```bash
  helm install myapp1 parentchart/ --atomic --set mychart2.replicaCount=2
  ```
