
# Homelab (Under Heavy Construction 🚧)

This repository contains my personal _in-progress_ homelab setup, intended to showcase my **DevOps** and **Cloud Native** learning journey. The setup uses **Ansible** to provision a lightweight Kubernetes cluster (via K3s) and **Flux** (with Kustomize) to manage application and infrastructure deployments in a GitOps manner. 

---

## Table of Contents

1. [Architecture Overview](#architecture-overview)  
2. [Folder Structure](#folder-structure)  
   - [ansible-k8s](#ansible-k8s)  
   - [apps](#apps)  
   - [clusters](#clusters)  
   - [database](#database)  
   - [infrastructure](#infrastructure)  
3. [Key Technologies](#key-technologies)  
4. [Getting Started](#getting-started)  
   - [Prerequisites](#prerequisites)  
   - [Installation & Usage](#installation--usage)  
5. [Current Status & Roadmap](#current-status--roadmap)  
6. [License](#license)  
7. [Contact](#contact)  

---

## Architecture Overview

1. **Provisioning with Ansible**  
   - Ansible playbooks in [ansible-k8s/](ansible-k8s/) bootstrap the underlying servers, install K3s, and set up Tailscale networking.  
   - Additional roles configure each node for Longhorn and other core capabilities.

2. **K3s as Lightweight Kubernetes**  
   - K3s is installed on a single master node (`captain`) and multiple worker nodes (`ahoy1`, `ahoy2`).  
   - A secure overlay network (Tailscale) is used for remote access and cluster management.

3. **GitOps with Flux & Kustomize**  
   - Flux watches this Git repository and automatically applies changes to the cluster.  
   - Kustomize overlays handle environment-specific configurations in [apps/the-big-ship/](apps/the-big-ship/), [database/the-big-ship/](database/the-big-ship/), and [infrastructure/the-big-ship/](infrastructure/the-big-ship/).

4. **Storage & Persistent Volumes**  
   - [Longhorn](https://longhorn.io/) provides highly available storage within the cluster.  
   - Some resources also demonstrate usage of [NFS-based PVs](apps/base/homepage/persistent_storage.yaml).

5. **Applications & Services**  
   - **Blog** (Hugo-based container)  
   - **Homepage** (a dynamic dashboard)  
   - **Vaultwarden** (Bitwarden alternative)  
   - **Immich** (photo management)  
   - **PostgreSQL** clusters with [CloudNativePG](https://cloudnative-pg.io/)  
   - **Monitoring** via Prometheus & Grafana  
   - **Traefik** as the Ingress Controller

---

## Folder Structure

```plaintext
.
├── ansible-k8s
│   ├── common/      # Common role tasks for all servers (installing curl, etc.)
│   ├── master/      # Master node role (installs K3s server, Tailscale, etc.)
│   ├── worker/      # Worker nodes role (K3s agents)
│   ├── longhorn/    # Role configuring node prerequisites for Longhorn
│   ├── inventory.ini # Ansible inventory
│   ├── setup_k3s_cluster.yaml  # Main playbook to set up K3s cluster
│   └── setup-longhorn.yaml     # Playbook to configure Longhorn cluster-wide
├── apps
│   ├── base/        # Base definitions for each application (Deployment, Service, Ingress)
│   └── the-big-ship/ # Environment-specific Kustomize overlays
├── clusters
│   └── the-big-ship/ # Flux Kustomizations for apps, databases, and infrastructure
├── database
│   ├── base/        # Base definitions for CloudNativePG clusters
│   └── the-big-ship/ # Overlays to deploy database instances for various apps
└── infrastructure
    ├── base/        # HelmRelease definitions (cert-manager, traefik, longhorn, etc.)
    └── the-big-ship/ # Overlays for production / main cluster environment
```

### ansible-k8s

- **common/**  
  Contains tasks shared by all hosts (e.g., installing `curl`).  
- **master/**  
  Installs and configures the K3s server on the master node along with Tailscale.  
- **worker/**  
  Joins worker nodes to the K3s cluster.  
- **longhorn/**  
  Ensures kernel modules/services required by Longhorn are in place on each node.  
- **inventory.ini**  
  Specifies Ansible host groups for `master` and `workers`.  
- **setup_k3s_cluster.yaml**  
  Main entry point to install K3s on master and workers.  
- **setup-longhorn.yaml**  
  Installs and configures Longhorn across the cluster.

### apps

- **base/**  
  Kustomize bases for each application (e.g., Blog, Homepage, Vaultwarden, etc.).  
- **the-big-ship/**  
  Overlays referencing `base` folders, often patching or adding configs (e.g., `homepage/configmap.yaml`) for production or a specific environment.

### clusters

- **the-big-ship/**  
  Flux Kustomization manifests that watch and apply changes from `apps/the-big-ship`, `database/the-big-ship`, and `infrastructure/the-big-ship`. Includes:
  - **apps-kus.yaml** & **database-kus.yaml**  
    Kustomizations to deploy apps and databases.  
  - **infrastructure-kus.yaml**  
    Kustomization to deploy cluster infrastructure (cert-manager, traefik, etc.).

### database

- **base/**  
  Base configs for CloudNativePG clusters, including: 
  - **playground/**  
    Example PostgreSQL cluster for testing purposes.
  - **vaultwarden-db/**  
    PostgreSQL cluster for Vaultwarden.
- **the-big-ship/**  
  Environment overlays for each database cluster (patching or customizing base resources).

### infrastructure

- **base/**  
  HelmRelease definitions for core services, such as:
  - **cert-manager**  
  - **cloudnative-pg**  
  - **longhorn**  
  - **monitoring** (Prometheus & Grafana)  
  - **traefik**  
  - **external-secrets**
  - **renovate**
- **the-big-ship/**  
  Kustomize overlays that adapt the base HelmRelease definitions to the environment (e.g., production vs. staging).

---

## Key Technologies

- **K3s (Lightweight Kubernetes)**
- **Ansible** (Provisioning and Configuration Management)
- **Flux** (GitOps Continuous Delivery)
- **Kustomize** (Kubernetes Native Configuration Management)
- **Longhorn** (Container-Native Distributed Storage)
- **CloudNativePG** (PostgreSQL Operator for Kubernetes)
- **Traefik** (Ingress Controller)
- **Prometheus & Grafana** (Monitoring Stack)
- **External Secrets** (K8s Operator for managing secrets)

---

## Getting Started

### Prerequisites

1. **Ansible Installed**  
   - For provisioning local or remote VMs/servers.
2. **SSH Access**  
   - Ensure you have passwordless SSH to your hosts or set up the necessary credentials in `inventory.ini`.
3. **K3s Requirements**  
   - A clean Linux OS on each target node (Ubuntu, Debian, etc.).
4. **Flux CLI (Optional)**  
   - Useful if you want to set up or modify GitOps workflows manually.
5. **Tailscale (Optional)**  
   - This repository is configured to use Tailscale for secure network overlay between cluster nodes.

### Installation & Usage

1. **Clone the Repository**  
   ```bash
   git clone https://github.com/ysonC/super-homelab.git
   cd super-homelab
   ```

2. **Update Inventory**  
   - In `ansible-k8s/inventory.ini`, set the IP addresses and SSH credentials for your master and worker nodes.

3. **Configure Secrets**  
   - Ansible vault is used in `ansible-k8s/secrets.yaml`. Provide or update your own secrets (tailscale auth key, etc.).  

4. **Run Ansible Playbooks**  
   - **Install K3s**:  
     ```bash
     ansible-playbook ansible-k8s/setup_k3s_cluster.yaml -i ansible-k8s/inventory.ini
     ```
   - **Setup Longhorn** (once cluster is running):  
     ```bash
     ansible-playbook ansible-k8s/setup-longhorn.yaml -i ansible-k8s/inventory.ini
     ```

5. **Bootstrap Flux**  
   - If Flux is not already installed, install it and point it to this repo. For example:  
     ```bash
     flux install
     flux create source git flux-system \
       --url=https://github.com/ysonC/super-homelab.git \
       --branch=main \
       --interval=1m \
       --export > flux-system/gotk-sync.yaml

     # Then commit and push changes so that Flux can reconcile
     git add -A && git commit -m "Bootstrap Flux" && git push
     ```
   - Flux will apply the Kustomizations found under `clusters/the-big-ship`, deploying infrastructure, apps, and databases accordingly.

---

## Current Status & Roadmap

- [x] **Basic K3s provisioning** using Ansible  
- [x] **Flux** GitOps pipeline set up  
- [x] **Traefik Ingress** with TLS certificates from cert-manager (using Cloudflare DNS challenge)  
- [x] **Longhorn** storage  
- [x] **Monitoring stack** (Prometheus, Grafana)  

### Next Steps

- [ ] Improve the CI/CD pipeline (GitHub Actions or GitLab CI for automated testing).  
- [ ] Add more advanced HelmRelease custom values for each application.  
- [ ] Integrate advanced security scanning (e.g., [Trivy](https://github.com/aquasecurity/trivy)).  
- [ ] Expand monitoring alerts and dashboards.  
- [ ] Add e2e tests for major services.

---

## License

[MIT License](LICENSE) © 2025 [Wyson Cheng]

---

## Contact

**Maintainer**: [@ysonC](https://github.com/ysonC)  
If you have suggestions or questions, feel free to open an issue or submit a PR.  

_Thank you for checking out my homelab setup!_
