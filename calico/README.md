# Calico CNI Configuration

This directory contains the `values.yaml` file used to configure **Calico**, the Container Network Interface (CNI) plugin for Kubernetes.

## Purpose

Calico is a fundamental component required for pod networking and network policy enforcement in the Kubernetes cluster. It is essential during the **early stages of cluster setup**, and therefore it is not deployed through GitOps (FluxCD), but instead via **Ansible** as part of the cluster bootstrap process.

## Deployment Method

Calico is deployed using an Ansible playbook to ensure it is available as soon as the control plane is initialized. This approach avoids dependency on the GitOps pipeline, which itself depends on a functioning CNI to work properly.

## Configuration Details

The `values.yaml` file defines:

- **CIDR range** and encapsulation method (`VXLAN`)
- **Tolerations** and **node selectors** to ensure deployment on all Linux nodes
- **Directories** for CNI binaries and configs
- **Systemd-based cgroup driver** to match the container runtime (containerd)
- **Typha component tolerations** for HA setups

## Important Notes

- Do not attempt to deploy this with Flux.
- Any modification to this file should be followed by a re-run of the Ansible playbook that handles CNI setup.
- This CNI setup assumes `containerd` with systemd as the cgroup driver (see `kubeletService.systemdCgroup`).
