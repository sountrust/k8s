## Flux Bootstrap & Kustomization Overview

This section documents the GitOps-based deployment architecture used in the `k8s/flux/` folder. It explains how the Flux CD controllers synchronize and manage both infrastructure components and application workloads across the Kubernetes cluster.

---

### 📦 Repository Structure

```
k8s/flux/
├── components.yaml
├── monacocloud-apps.yaml
├── monacocloud-config.yaml
├── flux-system/
│   ├── gotk-components.yaml
│   ├── gotk-sync.yaml
│   └── kustomization.yaml
```

- `components.yaml`: Installs the metrics-server components required for observability.
- `flux-system/`: Core Flux bootstrap files used by Ansible to initialize GitOps.
- `monacocloud-apps.yaml`: Contains GitRepository and Kustomization definitions for user applications.
- `monacocloud-config.yaml`: Contains GitRepository and Kustomization definitions for infrastructure services.

---

### 🚀 Flux Bootstrap Logic

Flux is bootstrapped by Ansible using the Flux CLI, with authentication and a remote GitLab or GitHub repository. The main components include:

- `flux-system/gotk-components.yaml`: Sets up Flux controllers.
- `flux-system/gotk-sync.yaml`: Synchronizes this repository with the cluster.
- `flux-system/kustomization.yaml`: Entry-point for recursive Flux reconciliation.

---

### 📁 Infrastructure Deployment via `monacocloud-config.yaml`

This file declares a `GitRepository` resource that tracks the infrastructure repository (e.g., `monacocloud_submodules/kubernetes`). It defines Kustomizations for the following services:

- `certmanager`: Issuer and certificate automation
- `traefik`: Ingress controller
- `prometheus`: Monitoring stack
- `helpmanager`: Internal app support services

Each Kustomization can depend on others using `dependsOn`, ensuring proper order of deployment.

---

### 🧩 Applications Deployment via `monacocloud-apps.yaml`

This file connects to the user workload repository (e.g., `kube-deploy`) and installs:

- `owncloud-manifests`
- `seeddms-manifests`
- `suitecrm-manifests`

These are auto-generated through `kube-deploy/builder/` and committed to Git. Flux automatically reconciles them.

---

### 🔐 Secrets & Auth

- Application repository (`kube-deploy`) uses: `flux-deploy-authentication`
- Infrastructure repository uses: `flux-system` secret

---

### 🔄 Reconciliation Interval

Each GitRepository and Kustomization has `interval: 1m0s`, meaning changes are detected and reconciled every 60 seconds.

---

### 📌 Notes

- GitOps ensures state reconciliation from Git to the cluster.
- Adding new applications simply requires creating manifests in the right folder.
- Pruning is enabled (`prune: true`), meaning deleted resources in Git will also be deleted in the cluster.
- Traefik must wait for cert-manager, Prometheus for Traefik, and HelpManager for Prometheus using `dependsOn`.

