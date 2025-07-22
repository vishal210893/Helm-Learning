# Helm Plugins

## Step 01: Introduction

Helm supports a powerful **plugin system** that extends its core functionality. Plugins can help with chart creation, visualization, dependency management, CI/CD pipelines, and more.

This guide focuses on:

* Installing and using **Helm Starter Plugin** (by Salesforce)
* Installing and using **Helm Dashboard Plugin** (by Komodor)

---

## Step 02: Install Helm Plugin

We begin by installing the **Helm Starter Plugin**, which simplifies working with Helm starter charts.

🔗 [Helm Starter Plugin GitHub](https://github.com/salesforce/helm-starter)

🔍 [plugin.yaml (Helm Starter)](https://github.com/salesforce/helm-starter/blob/master/plugin.yaml)

```bash
# List currently installed Helm plugins
helm plugin list

# Install Helm Starter Plugin
helm plugin install https://github.com/salesforce/helm-starter.git

# Confirm plugin installation
helm plugin list

# Get HELM_PLUGINS environment variable
helm env

# Observation:
# Look for the HELM_PLUGINS path, example:
# HELM_PLUGINS="/Users/kalyan/Library/helm/plugins"

# Go to Helm plugins directory
cd /Users/kalyan/Library/helm/plugins
ls
```

---

## Step 03: Play with Helm Starter Plugin

```bash
# Check plugin and its available subcommands
helm plugin list
helm <plugin-name> <sub-command>
helm starter list   # Lists available starter charts

# Fetch a sample starter chart from GitHub
helm starter fetch https://github.com/salesforce/helm-starter-istio.git

# Re-list starters to verify fetch
helm starter list
```

This allows you to manage, organize, and reuse starter charts across multiple charts in your organization.

---

## Step 04: Plugin Management Commands

Helm plugins can be updated or uninstalled as needed.

```bash
# Update a specific Helm plugin
helm plugin update <PLUGIN-NAME>
helm plugin update starter

# Uninstall a Helm plugin
helm plugin uninstall <PLUGIN-NAME>
helm plugin uninstall starter

# List plugins after uninstall
helm plugin list
```

---

## Step 05: Install Helm Releases (Sample Charts)

```bash
# Add custom Helm repo
helm repo add stacksimplify https://stacksimplify.github.io/helm-charts/
helm repo list

# Install a release named 'dev101'
helm install dev101 stacksimplify/mychart1 --atomic

# Upgrade the release with different replicaCount values
helm upgrade dev101 stacksimplify/mychart1 --atomic --set replicaCount=2
helm upgrade dev101 stacksimplify/mychart1 --atomic --set replicaCount=3

# Install another release
helm install dev102 stacksimplify/mychart2 --atomic

# List current Helm releases
helm list
```

---

## Step 06: (Optional) Install Helm Dashboard Plugin

🔗 [Helm Dashboard GitHub](https://github.com/komodorio/helm-dashboard)

🔗 [ArtifactHub Page](https://artifacthub.io/packages/helm-plugin/helm-dashboard/dashboard)

📄 [plugin.yaml (Helm Dashboard)](https://github.com/komodorio/helm-dashboard/blob/main/plugin.yaml)

```bash
# List Helm Plugins
helm plugin list

# Install Helm Dashboard Plugin
helm plugin install https://github.com/komodorio/helm-dashboard.git

# Start the dashboard server locally
helm dashboard
```

Once launched, the dashboard UI allows you to:

* Explore clusters and Helm releases
* View and diff revisions
* Visualize resources, manifests, values
* Manage charts visually without CLI commands

📌 Common Dashboard Sections:

* **Clusters**
* **Installed Charts**

    * Release: `dev101`, `dev102`
    * Revisions: Navigate revisions and view changes
    * Values, Notes, Manifests, Resources
* **Repositories**
* **Logout**

---

## Step 07: Uninstall Helm Releases

```bash
# Uninstall the releases created earlier
helm uninstall dev101
helm uninstall dev102
```

---

## Summary Table

| Plugin         | Purpose                                   | Command Example     |
| -------------- | ----------------------------------------- | ------------------- |
| helm-starter   | Manage and use Helm starter templates     | `helm starter list` |
| helm-dashboard | Visualize and manage Helm resources in UI | `helm dashboard`    |

With Helm plugins, you can dramatically improve your development workflow, automation, and visibility around Helm charts.
