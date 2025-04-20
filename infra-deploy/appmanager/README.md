# ☎️ HelpManager Service

The **HelpManager** is a custom-developed microservice used in the MonacoCloud infrastructure to manage customer interactions through the marketplace. It plays a crucial role in provisioning, pausing, or removing deployed applications (called "bricks") by interfacing with Kubernetes and GitOps automation layers.

---

## 🧩 Purpose

The HelpManager acts as a bridge between external customer events (e.g., purchases, user changes, deletions) and the underlying infrastructure. Its core functions include:

- Automatically deploying apps like SuiteCRM, SeedDMS, or OwnCloud when a user purchases a brick.
- Pausing deployments (e.g., setting replicas to 0) if a subscription lapses.
- Removing the entire stack and data if a brick is deleted.
- Sending alerting and feedback emails to users.
- Managing DNS records dynamically through external APIs.

This is made possible through a combination of:
- Containerized Java app with extensive config options.
- Integration with FluxCD, cert-manager, and Cassandra.
- Delegated execution of Git-controlled Bash scripts in `kube-deploy/builder`.

---

## ⚙️ Components

### 1. **HelpManager App Deployment**
A `Deployment` named `app` under the namespace `helperiance-manager`:
- Image: `helperiance/helperiance-manager:1.0.9`
- Manages environment config for Cassandra, Mail, DNS, and brick deployment.
- Secrets are injected from the `app-secrets` Kubernetes secret.
- Communicates with Cassandra and Kubernetes.
- Mounts `kube-deploy` repo internally to trigger bash automation.

### 2. **Cassandra DB Backend**
Deployment and `PersistentVolumeClaim` to persist HelpManager’s internal state.

- Image: `helperiance/cassandra:5.0.0`
- PVC: `cassandra-pvc` on NFS
- Service exposed on port `9042`

### 3. **Pod Auto-Recycling**
A CronJob named `delete-helperiance-pod-daily` under `helperiance-manager`:
- Deletes stale `helperiance-manager` pods daily at midnight to ensure freshness.
- Uses a dedicated `ServiceAccount` and RBAC role to restrict permissions.

---

## 🔐 Secrets & Sensitive Configuration

The app uses a `Secret` named `app-secrets` to manage all sensitive credentials and API tokens:
- Mail credentials (SMTP user/pass)
- DNS API credentials
- Cassandra DB access
- Memberpress API keys
- Git deployment token
- HelpManager API token for validation

These values are injected using `valueFrom.secretKeyRef` in the `Deployment` definition.

---

## 🚀 Integration with App Automation

HelpManager triggers bash scripts located inside the containerized volume at `/app/kube-deploy/builder`. These scripts are:

| App       | Deploy Script                        | Pause Script                       |
|-----------|--------------------------------------|------------------------------------|
| SuiteCRM  | `auto-deploy-suitecrm-manifest.sh`   | `auto-pause-suitecrm.sh`          |
| SeedDMS   | `auto-deploy-seeddms-manifest.sh`    | `auto-pause-seeddms.sh`           |
| OwnCloud  | `auto-deploy-owncloud-manifest.sh`   | `auto-pause-owncloud.sh`          |

The scripts generate Kubernetes manifests on-the-fly and commit them into the Flux-controlled Git repository. This allows for automated GitOps-based rollout of infrastructure changes.

PVC sizes are customizable per app (`crm.pvcSize`, `ged.pvcSize`, `backup.pvcSize`) and can be overridden per deployment.

---

## 🔁 Full Lifecycle

1. **Purchase event** → triggers a deploy script (generates manifests, Git commit/push).
2. **Pause event** → triggers pause script (replicas = 0).
3. **Delete event** → removes manifests from Git (Flux prunes resources).

---

## 🔧 Namespace & Resources

Namespace: `helperiance-manager`

Resources:
- `Deployment` for HelpManager app.
- `Deployment` for Cassandra.
- `CronJob` for daily pod cleanup.
- `PersistentVolumeClaim` for data persistence.
- `ServiceAccount`, `Role`, and `RoleBinding` for RBAC.
- `Service` exposing Cassandra internally.

---

## 🧪 Monitoring

The HelpManager app and Cassandra can be monitored using Prometheus/Grafana if their pods are annotated appropriately (not shown in current setup).
