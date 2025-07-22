# Helm Starters

## Step 01: Introduction

Helm Starter Charts are reusable chart templates that help developers scaffold new Helm charts quickly and consistently. These starter templates are extremely useful for organizations that follow standard Helm chart practices and want to avoid starting from scratch each time.

**Key Benefits:**

* Promote **reusability** and **consistency** across teams
* Enforce baseline standards (like naming conventions, annotations, structure)
* Simplify onboarding for new developers
* Embed standard dependencies, structure, and metadata

---

## Step 02: Helm Starter Charts Explained

### ❓ What are Starter Charts?

1. Just like regular Helm charts, but used **as templates** for creating new charts.
2. Contain templated content that will be customized upon chart creation.
3. Can include dependencies, default values, Kubernetes objects, and even sample `NOTES.txt`.

### 📁 Where Should They Be Stored?

Starter charts are placed under:

```
$HELM_DATA_HOME/starters/
```

You can find this path using:

```bash
helm env HELM_DATA_HOME
```

### ⚠️ Limitations

* The generated `Chart.yaml` will **overwrite** the original values from the starter chart.
* Version numbers and dependencies **do not get carried over** from the starter.

---

## Step 03: Create a Starter Chart (`mystarterchart`)

This chart will serve as the boilerplate template.

```bash
helm create mystarterchart
```

### 🎯 Clean and Customize It

Make the chart minimal and tailored:

**Inside `templates/`:**

* ❌ Delete: `tests/`, `hpa.yaml`, `ingress.yaml`, `serviceaccount.yaml`
* 🧹 Remove unused templates from `_helpers.tpl`
* 🛠 Update `deployment.yaml` and `service.yaml` to reflect a NodePort-based app

**`values.yaml`:**

* Set `service.type` to `NodePort` and `service.nodePort` to `31239`
* Set `image.repository` to:

  ```
  ghcr.io/stacksimplify/kubenginxhelm
  ```

**`Chart.yaml`:**

* Change version and appVersion to `1.0.0`
* Add a dependency:

  ```yaml
  dependencies:
  - name: mychart4
    version: "0.1.0"
    repository: "https://stacksimplify.github.io/helm-charts/"
  ```

**Download and Untar Subchart:**

```bash
cd mystarterchart/charts
helm pull https://stacksimplify.github.io/helm-charts/mychart4-0.1.0.tgz --untar
```

---

## Step 04: Test Before Promoting to Starter

```bash
cd mystarterchart

helm lint
helm install myapp1 . --atomic

kubectl get pods
kubectl get svc
```

🧪 **Access Application:**

* Parent chart: [http://localhost:31239](http://localhost:31239)
* Subchart (mychart4): Use NodePort from `kubectl get svc`

```bash
helm uninstall myapp1
```

---

## Step 05: Convert to a Starter Chart

Replace all hardcoded chart references with `<CHARTNAME>`:

* `_helpers.tpl`
* `deployment.yaml`, `service.yaml`
* `NOTES.txt`
* `Chart.yaml`
* `values.yaml` (just in comments or description)

These will be dynamically replaced with the actual chart name when you create new charts from this starter.

---

## Step 06: Move to HELM\_DATA\_HOME/starters

```bash
# Get the Helm data directory
helm env HELM_DATA_HOME

# Example: /Users/kalyan/Library/helm
cd /Users/kalyan/Library/
mkdir -p helm/starters
cp -r mystarterchart helm/starters/
```

---

## Step 07: Generate a Chart Using Starter

```bash
cd MYCHARTS
helm create mychart9 --starter=mystarterchart
```

### ✅ What Happens:

* A new chart `mychart9` is created using the starter layout
* `Chart.yaml` is regenerated with:

    * `name: mychart9`
    * `version: 0.1.0`
    * `appVersion: 0.1.0`
* Existing `.tgz` in `charts/` gets regenerated from source starter's unpacked chart

Update `Chart.yaml` to match the Docker image tag:

```yaml
appVersion: "0.3.0"
```

---

## Step 08: Install the New Chart

```bash
cd MYCHARTS/mychart9

helm lint
helm install myapp901 .
```

Check everything:

```bash
kubectl get pods
kubectl get svc
```

🎯 **Access Applications**

* `mychart9`: [http://localhost:31239](http://localhost:31239)
* `mychart4`: [http://localhost](http://localhost):<nodeport-from-kubectl>

```bash
helm uninstall myapp901
```

---

## Summary

| Step | Description                                                               |
| ---- | ------------------------------------------------------------------------- |
| ✅ 01 | Starter Charts are reusable Helm chart templates                          |
| ✅ 02 | Place them in `$HELM_DATA_HOME/starters`                                  |
| ✅ 03 | Customize a base chart and convert it to a starter                        |
| ✅ 04 | Create charts from starter using `helm create <name> --starter=<starter>` |
| ✅ 05 | Starter’s `Chart.yaml` gets overridden on new chart creation              |
| ✅ 06 | Subcharts are packaged into `.tgz` files automatically                    |

Helm starters are ideal for standardizing chart development across teams and projects 🚀.
