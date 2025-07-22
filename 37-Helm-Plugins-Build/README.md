# Helm Plugin Development

## Step-01: Introduction

Helm provides a powerful [plugin system](https://helm.sh/docs/topics/plugins/#building-plugins) that allows users to extend its CLI with custom commands. These plugins can be simple shell scripts, binaries, or even platform-specific commands.

In this guide, we will:

* Build and test **three simple custom plugins**:

    * `myplugin1`: Basic env command
    * `myplugin2`: Platform-aware commands
    * `myplugin3`: Shell script-based plugin

---

## Step-02: Create `myplugin1` – Print Helm Environment Variables

### Plugin Metadata (`plugin.yaml`)

```yaml
name: "myplugin1"
version: "0.1.0"
usage: "Prints Helm Environment Variables"
description: |-
  Prints Helm Environment Variables
command: "env"
```

### Plugin Setup & Execution

```bash
# List existing plugins
helm plugin list

# Install plugin from directory
helm plugin install myplugin1/

# Re-list plugins to confirm installation
helm plugin list

# Execute plugin
helm myplugin1

# Expected Output:
# Prints all current Helm-related environment variables
```

---

## Step-03: Create `myplugin2` – Platform-Aware Commands

### Plugin Metadata (`plugin.yaml`)

```yaml
name: "myplugin2"
version: "0.1.0"
usage: "helm myplugin2"
description: "Print Helm plugin directory"
command: echo my helm plugin directory is $HELM_PLUGINS default command
platformCommand:
  - os: linux
    arch: i386
    command: "echo my helm plugin directory is $HELM_PLUGINS os is linux i386"
  - os: linux
    arch: amd64
    command: "echo my helm plugin directory is $HELM_PLUGINS os is linux amd64"
  - os: windows
    arch: amd64
    command: "echo my helm plugin directory is $HELM_PLUGINS os is windows amd64"
```

### Plugin Setup & Execution

```bash
# Install the plugin
helm plugin install myplugin2/

# Run the plugin
helm myplugin2

# Observation:
# Since the current system is MacOS (not listed in platformCommand), 
# it will fallback and execute the default command.
```

---

## Step-04: Create `myplugin3` – Shell Script-based Plugin

### Plugin Metadata (`plugin.yaml`)

```yaml
name: "myplugin3"
version: "0.1.0"
usage: "helm myplugin3"
description: "Print Helm plugin directory using script app.sh"
command: "$HELM_PLUGIN_DIR/app.sh"
```

### Plugin Script (`app.sh`)

```bash
#!/bin/sh
echo "my helm plugin directory is $HELM_PLUGINS from SHELL SCRIPT"
```

> 📌 Make sure `app.sh` has executable permissions:

```bash
chmod +x app.sh
```

### Plugin Setup & Execution

```bash
# Install the plugin
helm plugin install myplugin3/

# Run the plugin
helm myplugin3

# Expected Output:
# my helm plugin directory is <path> from SHELL SCRIPT
```

---

## Step-05: Uninstall Helm Plugins

```bash
# Uninstall one or more plugins
helm plugin uninstall myplugin1
helm plugin uninstall myplugin2
helm plugin uninstall myplugin3

# Verify removal
helm plugin list
```

---

## Summary

| Plugin Name | Description                               | Type               | Custom Logic                     |
| ----------- | ----------------------------------------- | ------------------ | -------------------------------- |
| myplugin1   | Prints Helm environment variables         | Simple env command | CLI `env`                        |
| myplugin2   | Demonstrates platform-specific logic      | Platform-aware     | `platformCommand` block fallback |
| myplugin3   | Executes a custom shell script (`app.sh`) | Script-based       | Uses `$HELM_PLUGIN_DIR` variable |

By using these examples, you can explore Helm plugin capabilities and begin building more sophisticated CLI tooling customized to your development or DevOps needs.
