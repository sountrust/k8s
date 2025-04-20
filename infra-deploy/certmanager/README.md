# cert-manager – Flux-Managed Deployment

This directory contains the configuration necessary to deploy and manage **cert-manager** inside the Kubernetes cluster using **FluxCD**. It ensures automatic TLS certificate provisioning using Let's Encrypt.

## 📁 Structure

```
certmanager/
├── cert-manager.yaml            # Downloaded cert-manager manifests (v1.17.0) with edits
├── clusterissuer-acme.yaml      # ClusterIssuer to request TLS certificates from Let's Encrypt
└── kustomization.yaml           # Flux Kustomization file to apply resources
```

## 🔧 What’s Included

### 1. `cert-manager.yaml`
- This is a local copy of the official manifest from the [cert-manager GitHub release](https://github.com/cert-manager/cert-manager/releases/tag/v1.17.0).
- Downloaded via:
  ```bash
  wget https://github.com/cert-manager/cert-manager/releases/download/v1.17.0/cert-manager.yaml
  ```
- It includes all necessary CRDs, deployments, RBAC, and services.
- **Customized** to align with our cluster setup (e.g., namespace, labels, or image registry if required).

### 2. `clusterissuer-acme.yaml`
- Configures a `ClusterIssuer` for Let's Encrypt ACME HTTP01 challenge using Traefik as the ingress controller.
- Uses production server endpoint:
  ```yaml
  server: https://acme-v02.api.letsencrypt.org/directory
  ```

### 3. `kustomization.yaml`
- Defines which resources to apply via Flux.
- Lists both the main cert-manager manifest and the cluster issuer.
- Example:
  ```yaml
  resources:
    - cert-manager.yaml
    - clusterissuer-acme.yaml
  ```

## 🚀 Flux Integration
This folder is applied via the Flux `Kustomization` defined in the main `k8s/flux` structure under the name `certmanager`.

It is part of a dependency chain and will be applied **before Traefik**, as Traefik uses certificates issued by cert-manager.

## 🛡️ TLS Issuance Workflow
1. cert-manager installs and initializes from the local manifest.
2. The ClusterIssuer is applied and registers with Let’s Encrypt.
3. Any service requiring a TLS certificate refers to this issuer (`acme-issuer`).
4. cert-manager uses Traefik ingress with HTTP-01 challenge to validate domain ownership.

---

Ensure the domain used is publicly resolvable and ingress routes are functional before requesting certificates.
