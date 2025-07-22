# Helm Subcharts - Dependency Command

## Step-01: Introduction

In this section, you'll explore how Helm handles subcharts through its dependency mechanism. We'll cover:

* Declaring chart dependencies in `Chart.yaml`
* Managing them using `helm dependency` commands
* Using semantic version constraints effectively
* Understanding differences between using `@REPO` vs direct repository URLs

---

## Step-02: Create Parent Chart

```bash
helm create parentchart
```

This generates a basic Helm chart structure. You'll enhance it by adding dependencies on other charts.

---

## Step-03: Define Dependencies in `Chart.yaml`

Edit `parentchart/Chart.yaml` and append the following under `dependencies:`:

```yaml
dependencies:
  - name: mychart1
    version: "0.1.0"
    repository: "https://stacksimplify.github.io/helm-charts/"
  - name: mychart2
    version: "0.4.0"
    repository: "https://stacksimplify.github.io/helm-charts/"
  - name: mysql
    version: "9.9.0"
    repository: "https://charts.bitnami.com/bitnami"
```

These are subcharts that will be downloaded into `charts/` directory of the parent chart.

---

## Step-04: Run Helm Dependency Commands

### View Dependencies

```bash
helm dependency list parentchart
```

**Expected Output:** Dependencies listed with status `missing`

### Verify Empty `charts/` Folder

```bash
ls parentchart/charts
```

Should be empty initially.

### Update Dependencies

```bash
helm dependency update parentchart/
```

This downloads specified chart packages and creates:

* `charts/mychart1-0.1.0.tgz`
* `charts/mychart2-0.4.0.tgz`
* `charts/mysql-9.9.0.tgz`
* `Chart.lock` file in `parentchart/`

### Review Lock File

```bash
cat parentchart/Chart.lock
```

### Confirm Dependencies are Installed

```bash
helm dependency list parentchart/
```

Status should now show `ok`.

---

## Step-05: Helm Dependency Version Constraints

### Helm Chart Version Format

```
Major.Minor.Patch
Example: 9.10.8
```

### Version Constraint Operators

```yaml
version: "= 9.10.8"
version: "!= 9.10.8"
version: ">= 9.10.8"
version: "<= 9.10.8"
version: "> 9.10.8"
version: "< 9.10.8"
version: ">= 9.10.8 < 9.11.0"
```

### Caret (^) - Major Range Constraint

```yaml
^9.10.1  ≈ >= 9.10.1, < 10.0.0
^9.10.x  ≈ >= 9.10.0, < 10.0.0
^9.x     ≈ >= 9.0.0, < 10.0.0
```

### Tilde (\~) - Patch/Minor Constraint

```yaml
~9.10.1  ≈ >= 9.10.1, < 9.11.0
~9.10    ≈ >= 9.10.0, < 9.11.0
~9       ≈ >= 9.0.0,  < 10.0.0
```

### Example Constraint in `Chart.yaml`

```yaml
dependencies:
  - name: mysql
    version: "~9.9.0"
    repository: "https://charts.bitnami.com/bitnami"
```

### Apply Changes

```bash
helm dependency update parentchart/
```

---

## Step-06: `helm dependency build`

```bash
helm dependency build parentchart/
```

* Reconstructs the `charts/` directory based on `Chart.lock`
* Doesn't negotiate version constraints again; simply uses what’s locked
* Useful in CI/CD when `Chart.lock` is checked into version control

---

## Step-07: `@REPO` vs `repository: URL`

### Recommended: Use Full URL

```yaml
dependencies:
  - name: mysql
    version: ">=9.10.8"
    repository: "https://charts.bitnami.com/bitnami"
```

### Not Recommended: Using `@REPO`

```yaml
dependencies:
  - name: mysql
    version: ">=9.10.8"
    repository: "@bitnami"
```

To use `@bitnami`, it must be added first:

```bash
helm repo add bitnami https://charts.bitnami.com/bitnami
helm repo update
```

### Cleanup and Re-try

```bash
rm -rf parentchart/charts/*
rm parentchart/Chart.lock
helm dependency update parentchart/
```

Both forms will work, but using explicit URLs is best for portability and automation, especially in CI pipelines or when `helm repo add` isn't guaranteed.

---

> Tip: Always commit `Chart.lock` into version control when using `helm dependency build` in CI/CD to ensure consistent builds across environments.
