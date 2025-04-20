# GitLab Agent (Optional Integration)

This folder holds the configuration manifests required to deploy the GitLab Kubernetes Agent using Flux. Although this component is present in the repository, **it is not currently deployed** in the cluster by default.

---

## 📁 Structure

```
k8s/infra-deploy/gitlab/
├── gitlab-agent-helmrelease.yaml    # Helm release definition for GitLab Agent
├── gitlab-agent-namespace.yaml      # Namespace and Helm repository
```

---

## ⚙️ Purpose

This setup enables CI/CD integration between GitLab and the Kubernetes cluster through the GitLab Agent (Agentk). It facilitates secure communication for pipeline jobs, deployments, and GitOps workflows.

However, the GitLab agent resources are **not automatically deployed** because this directory is **not referenced** in the Flux source configuration (`your-cloud-config.yaml`). This means it's opt-in and can be enabled manually when needed.

---

## 🧭 How It Works (When Activated)

- Defines a `gitlab` namespace.
- Adds the official GitLab Helm repository.
- Installs the `gitlab-agent` Helm chart.
- Configures the agent to connect to GitLab using `kasAddress` and a referenced `gitlab-agent-token` secret.

---

## 🚀 Enable GitLab Agent (If Needed)

To activate this configuration:

1. Edit `k8s/flux/your-cloud-config.yaml` and add:

```yaml
- ./infra-deploy/gitlab
```

under the appropriate `Kustomization.spec.path` field.

2. Commit and push the change to trigger Flux reconciliation.

---

## 🧹 Cleanup

To remove GitLab Agent resources (if ever deployed):

```bash
kubectl delete namespace gitlab
```

---

## 🔒 Notes

- The agent expects a Kubernetes secret named `gitlab-agent-token` to exist in the `gitlab` namespace.
- This is useful for scenarios where GitLab CI/CD pipelines need direct cluster access.
- Since it's not referenced by Flux currently, no resources are deployed unless explicitly configured.

