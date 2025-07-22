Certainly! Here's the fully formatted documentation for your **Helm Masterclass: 50 Practical Demos for Kubernetes DevOps** in a consistent style:

---

# Helm Masterclass: 50 Practical Demos for Kubernetes DevOps

## Course Details

* **Title:** Helm Masterclass: 50 Practical Demos for Kubernetes DevOps
* **Subtitle:** Create, Develop, Install, Upgrade, Rollback, Package, and Publish Helm Charts with step-by-step practical demos.

---

## Course Modules

1. Install Docker Desktop and HelmCLI
2. Helm Install
3. Helm Upgrade with set option
4. Helm Upgrade with Chart Versions
5. Helm Uninstall Keep History
6. Helm Install Generated Name
7. Helm Install Atomic
8. Helm with Namespaces
9. Helm Override Values
10. Helm Chart Structure
11. Helm Dev BuiltIn Objects
12. Helm Dev Basics
13. Helm Dev If Else EQ
14. Helm Dev If Else AND BOOLEAN
15. Helm Dev If Else OR
16. Helm Dev If Else NOT
17. Helm Dev WITH
18. Helm Dev WITH If Else
19. Helm Dev Variables
20. Helm Dev Range List
21. Helm Dev Range Dict
22. Helm Dev Named Templates
23. Helm Dev Printf Function
24. Helm Dev call template in template
25. Helm Create and Package Chart
26. Helm Dependency
27. Helm Dependency Alias
28. Helm Dependency Condition
29. Helm Dependency Condition Alias
30. Helm Dependency Tags
31. Helm Dependency Override Subchart Values
32. Helm SubChart Global Values
33. Helm Dependency Import Values Explicit
34. Helm Dependency Import Values Implicit
35. Helm Starters
36. Helm Plugins
37. Helm Plugins Build
38. Helm Hooks
39. Helm Hooks Delete Policy
40. Helm Hook Weights
41. Helm Tests
42. Helm Resource Policy
43. Helm Sign and Verify Charts
44. Helm Repo on GitHub
45. Integrate with ArtifactHub
46. Helm Values Validate with JSON Schema

---

## What Will Students Learn in This Course?

* Master all 24 Helm commands and their subcommands/flags through practical hands-on demos.
* Develop Helm charts using 13 focused development exercises.
* Understand and apply flow control structures: `if-else`, `with`, and `range`, along with operators and functions like `eq`, `and`, `or`, `not`, `default`, and `quote`.
* Create, install, upgrade, rollback, and uninstall Helm charts.
* Implement Helm dependencies using 9 demos covering aliases, conditions, tags, global values, and value imports.
* Explore Helm concepts like starters, plugins, hooks, tests, resource policies, and values schema validations.
* Sign and verify Helm charts securely.
* Host your own Helm repository on GitHub and integrate it with Artifact Hub.

---

## Prerequisites

* **Kubernetes proficiency is required.**
* You must be comfortable working with Kubernetes clusters and resources to follow along with the hands-on labs and examples.

---

## Who Should Take This Course?

* Students who have completed:

    * AWS EKS
    * Azure AKS
    * Google GKE Kubernetes courses
* Infrastructure Architects
* System Administrators
* Developers
* DevOps Engineers

Anyone planning to **master Helm** in real-world production environments.

---

## GitHub Repositories Used in This Course

* [helm-masterclass](https://github.com/stacksimplify/helm-masterclass)
* [helm-charts](https://github.com/stacksimplify/helm-charts)
* [helm-charts-repo](https://github.com/stacksimplify/helm-charts-repo)
* [Course Presentation](https://github.com/stacksimplify/helm-masterclass/course-presentation/)

**Important Note:**
Please **FORK** the above repositories into your own GitHub account and use them throughout the course for practical exercises and reference.

---

```
helm install myapp1 stacksimplify/mychart1
kubectl port-forward service/myapp1-nodeport-service 31231:80
```