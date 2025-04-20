# Prometheus Monitoring – Infra Deploy Layer

This folder sets up monitoring capabilities for the Kubernetes clusters using the Prometheus stack via Flux and Helm. It includes the deployment of Prometheus, Grafana, alerting capabilities, storage setup, and integration with other key tools such as Traefik and Cert-Manager.

---

## 📁 Files Overview

### `prometheus-namespace.yaml`
Defines a dedicated Kubernetes namespace (`monitoring`) to isolate all monitoring-related resources.

---

### `prometheus-helm-repo.yaml`
Defines the Helm repository source used to install the Prometheus stack:
- Source: https://prometheus-community.github.io/helm-charts
- Registered under the `flux-system` namespace
- Interval: syncs every 1 minute

---

### `prometheus-helmrelease.yaml`
Main file for deploying the Prometheus stack with custom configurations:

- **Chart:** `kube-prometheus-stack` from the `prometheus-community` Helm repo
- **Namespace:** `monitoring`
- **Storage:** Uses a `PersistentVolumeClaim` backed by `nfs-client`, with 1Gi of space
- **Prometheus Customization:**
  - Retention of 5 days
  - WAL compression enabled
  - No out-of-order samples allowed
  - Scrape configs:
    - Traefik (based on pod annotations)
    - Cert-Manager
    - FluxCD
  - Custom metrics filtering for `gotk_resource_info`
- **Grafana:**
  - Ingress enabled
  - Uses Traefik ingress class
  - TLS enabled via the `grafana` certificate
  - Exposed on `dev.dashboard.your.site`

---

### `grafana-certificate.yaml`
Defines a TLS certificate for Grafana via Cert-Manager:
- Domain: `dev.dashboard.your.site`
- Issuer: `acme-issuer` (defined in the `certmanager` module)
- Secret: `grafana` (used by the HelmRelease ingress)

---

### `kustomization.yaml`
Standard Flux `Kustomization` manifest for deploying this folder:
```yaml
resources:
  - prometheus-namespace.yaml
  - prometheus-helm-repo.yaml
  - prometheus-helmrelease.yaml
  - grafana-certificate.yaml
```

This allows Flux to recursively sync all Prometheus-related resources in the proper order.

---

## 🔄 Integration Flow
- **Flux** installs the Helm chart based on the HelmRelease
- **Grafana** and **Prometheus** are installed with predefined scrape targets
- **TLS** is enabled for Grafana via Cert-Manager and ACME
- **Ingress** is managed by Traefik (installed in another `infra-deploy` module)

---

## 🧠 Notes
- Metrics from Traefik, Cert-Manager, and Flux are scraped automatically
- Customize storage, scrape configs, and retention as needed
- Consider adding alerts and dashboards via Grafana provisioning if required

