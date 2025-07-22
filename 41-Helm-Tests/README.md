# Helm Tests

## Step-01: Introduction

Helm allows you to define and run tests for your chart using the `helm test` command. These tests are Kubernetes pods annotated with `helm.sh/hook: test`, designed to verify that a release is working as expected after deployment.

---

## Step-02: Create Helm Chart and Install Release

```t
# Create a new Helm chart
helm create mydemoapp

# Install a release from the chart
helm install myapp101 mydemoapp/

# Verify the release
helm list
```

---

## Step-03: Review the Helm Test YAML File

By default, Helm generates a test pod in the following path:

* **File:** `mydemoapp/templates/tests/test-connection.yaml`

This file defines a basic test hook using a `Pod` resource:

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: "{{ include "mydemoapp.fullname" . }}-test-connection"
  labels:
    {{- include "mydemoapp.labels" . | nindent 4 }}
  annotations:
    "helm.sh/hook": test
spec:
  containers:
    - name: wget
      image: busybox
      command: ['wget']
      args: ['{{ include "mydemoapp.fullname" . }}:{{ .Values.service.port }}']
  restartPolicy: Never
```

**Key Detail:**
The annotation `"helm.sh/hook": test` marks this pod as a test hook. When `helm test` is executed, Helm launches this pod and waits for it to complete.

---

## Step-04: Run Helm Test and Verify

```t
# Check for running pods
kubectl get pods

# Execute Helm test for the release
helm test myapp101

# Review pods again
kubectl get pods
```

**Observation:**

* A pod named `myapp101-mydemoapp-test-connection` is created.
* The pod will move to a `Completed` status if the test passes.

```t
# Example output
NAME: myapp101
LAST DEPLOYED: Tue Jul 22 23:10:00 2025
NAMESPACE: default
STATUS: deployed
REVISION: 1
TEST SUITE:     myapp101-mydemoapp-test-connection
Last Started:   Tue Jul 22 23:10:02 2025
Last Completed: Tue Jul 22 23:10:15 2025
Phase:          Succeeded
```

---

## Step-05: Uninstall Helm Release and Clean Up

```t
# Uninstall the Helm release
helm uninstall myapp101

# Verify the release is removed
helm list

# Clean up any remaining pods
kubectl get pods
```

Using `helm test` is a practical way to validate deployments automatically and catch potential issues early in CI/CD pipelines or during manual testing.
