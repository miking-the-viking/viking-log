# [Helm](https://helm.sh/)

Helm is a tool for managing Kubernetes charts. Charts are packages of pre-configured Kubernetes resources.

Helm can be used to:

- Find and use popular software packaged as Kubernetes charts
- Share applications as Kubernetes charts
- Create reproducible builds of Kubernetes applications
- Intelligently manage Kubernetes manifest files
- Manage releases of Helm packages

_Essentially, Helm can be likened to apt/yum/homebrew but for Kubernetes._

## References

- [Helm GitHub](https://github.com/kubernetes/helm)
- [Helm Docs](https://docs.helm.sh/)

## Nutshell Summary

- Helm has two parts: client (`helm`) and server (`tiller`)
- Tiller runs inside of the Kubernetes cluster, nad manages releases (installations) of charts
- Helm runs on a local device (laptop), CI/CD, or wherever desired
- Charts are Helm packages that contain a minimum of two items:
  - A description of the package `Chart.yaml`
  - 1...N templates containing Kubernetes manifest files
- Charts can be stored on a disk, or fetched from a remote chart repository (Debian, RedHat, ...)

...
...