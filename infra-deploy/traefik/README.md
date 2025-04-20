# Traefik Ingress Controller – Flux Deployment

This folder contains the FluxCD configuration to deploy the **Traefik ingress controller** using Helm in the Kubernetes cluster.

---

## 📦 Files

- `traefik-helm-repo.yaml`: Registers the official Traefik Helm chart repository with Flux.
- `traefik-helmrelease.yaml`: Defines the actual HelmRelease resource to install and configure Traefik via Flux.

---

## 🚀 What It Does

This deployment sets up **Traefik** as the default ingress controller with fixed node ports and TLS support, making it suitable for external HTTP/HTTPS routing in the Kubernetes cluster.

---

## 🛠️ Configuration Breakdown

### `traefik-helm-repo.yaml`

This file tells Flux to continuously poll the [Traefik Helm chart repository](https://traefik.github.io/charts) for updates every minute.

```yaml
apiVersion: source.toolkit.fluxcd.io/v1
kind: HelmRepository
metadata:
  name: traefik
  namespace: flux-system
spec:
  interval: 1m0s
  url: https://traefik.github.io/charts
```

### `traefik-helmrelease.yaml`

This HelmRelease deploys Traefik with:

- Exposed HTTP (80) and HTTPS (443) ports via fixed `NodePort` values:
  - Port 80 → NodePort `30080`
  - Port 443 → NodePort `30443`
- TLS enabled on the HTTPS endpoint.
- Traefik’s default ingress class enabled.
- Metrics exposed for Prometheus via pod annotations.
- Logging set to `DEBUG`.

The release is configured to run in the `default` namespace, but the Helm chart itself is fetched from the `flux-system` namespace.

Key config:
```yaml
service:
  type: NodePort
  enabled: true

ports:
  web:
    nodePort: 30080
    redirectTo:
      port: websecure
  websecure:
    nodePort: 30443
    tls:
      enabled: true
```

---

## 🔄 Dependency

This deployment is part of the Flux `infra-deploy` stack and must be applied **after** `cert-manager`, as it relies on TLS for ingress routes.

---

## 📎 Notes

- Traefik is essential for routing traffic to all exposed applications in this cluster.
- Flux monitors this directory and applies changes automatically.
- The `traefik-helmrelease.yaml` is version-pinned to `28.1.0-beta.3` for consistency.

---

## 🧩 Related Flux Resources

- Namespace: `flux-system`
- GitRepository: Defined in `your-cloud-configs` under `k8s/flux/your-cloud-config.yaml`
- Depends on: `certmanager
