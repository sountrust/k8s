```markdown
# 🧠 Kubernetes GitOps Configuration

This repository defines the GitOps structure for deploying and managing both the **infrastructure** and **application workloads** of a kubernetes cluster using [Flux CD](https://fluxcd.io/).

Everything is declarative and driven by Git, ensuring full version control and automation of the Kubernetes state.

---

## 📁 Structure

```bash
k8s/
├── flux/                # Flux bootstrapping: Git sources + Kustomizations
│   ├── components.yaml
│   ├── your-cloud-config.yaml
│   ├── your-cloud-apps.yaml
│   └── flux-system/
│       ├── gotk-components.yaml
│       ├── gotk-sync.yaml
│       └── kustomization.yaml

├── infra-deploy/        # Core infrastructure components
│   ├── certmanager/
│   ├── traefik/
│   ├── prometheus/
│   ├── gitlab/
│   └── appmanager/

├── calico/              # Calico CNI configuration (used early by Ansible)
│   └── values.yaml

├── flux/                # GitRepository and Kustomization definitions
├── README.md
```

---

## 🚀 How It Works

Flux bootstraps two key Git repositories:

1. **your-cloud-configs** → this repository  
   Contains the _infrastructure layer_ (cert-manager, traefik, prometheus, app-manager…)

2. **your-cloud-apps** → [kube-deploy](https://github.com/sountrust/app-deploy)  
   Contains the _application layer_ (ownCloud, SeedDMS, SuiteCRM…)  
   These apps are generated/managed dynamically via the `app-manager`.

---

## 🔧 Early Setup via Ansible

> Not all components are deployed by Flux.

Some tools (like **Calico CNI**) are deployed in the early provisioning stages via **Ansible**, due to network prerequisites. These include:

- Calico (`k8s/calico/values.yaml`)
- Flux bootstrapping (`flux_conf.yaml`)
- Secrets, kubeconfig sync, etc.

---

## 🛠️ Subsystems

### 1. `infra-deploy/*`
Contains declarative configurations (mostly HelmReleases) for core infrastructure components:

- **certmanager/**  
  - `cert-manager.yaml`: downloaded and tweaked from upstream  
  - `clusterissuer-acme.yaml`: Let's Encrypt issuer (ACME)
  
- **traefik/**  
  - `traefik-helm-repo.yaml` + `traefik-helmrelease.yaml`: ingress controller
  
- **prometheus/**  
  - `prometheus-helmrelease.yaml`: kube-prometheus-stack
  - Includes scraping setup for: traefik, cert-manager, flux

- **app-manager/**  
  - Manages app lifecycle from marketplace subscriptions  
  - Calls bash scripts in `app-deploy/builder/*` to deploy/pause CRM, GED, backup (see docs)

- **gitlab/** *(optional)*  
  - GitLab Agent configuration — disabled by default
  - Can be added to Flux configs if CI integration is needed

---

## ⚙️ Application Delivery – `your-cloud-apps`

The [kube-deploy](https://github.com/sountrust/app-deploy) repo contains a set of bash builders that generate Kustomized app manifests based on client needs:

- **Apps supported**:
  - SuiteCRM
  - SeedDMS
  - OwnCloud

Flux continuously watches that repo and deploys any manifest pushed via the `app-manager`.

---

## 🧹 Deletion / Cleanup

To clean optional GitLab Agent:

```bash
kubectl delete namespace gitlab
```

---

## 🔐 Secrets

Secrets (like mail credentials, Cassandra, DNS, etc.) are injected via:

```bash
kubectl create secret generic app-secrets \
  --from-literal=mail.username="..." \
  --from-literal=app.api.token="..." \
  ...
```

These are referenced in `app-manager` and other deployments via `valueFrom: secretKeyRef`.

---

## ✍️ Contribution

> This repo is GitOps-first. All changes to infrastructure or apps **must be committed and pushed** to Git. Flux will then reconcile them automatically.

```bash
# Add changes
git add .

# Commit
git commit -m "Update app-manager config"

# Push
git push
```

---
```
