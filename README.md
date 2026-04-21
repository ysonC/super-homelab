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
   - [networking](#networking)
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

   - Ansible playbooks in [ansible-k8s/playbook/](ansible-k8s/playbook/) bootstrap the underlying servers, install K3s, and set up Tailscale networking.
   - Additional roles configure each node for Longhorn and other core capabilities.

2. **K3s as Lightweight Kubernetes**

   - K3s is installed on a single master node (`captain`) and multiple worker nodes (`ahoy1`, `ahoy2`).
   - A secure overlay network (Tailscale) is used for remote access and cluster management.

3. **GitOps with Flux & Kustomize**

   - Flux watches this Git repository and automatically applies changes to the cluster.
   - Kustomize overlays handle environment-specific configurations in [apps/the-big-ship/](apps/the-big-ship/), [database/the-big-ship/](database/the-big-ship/), and [infrastructure/the-big-ship/](infrastructure/the-big-ship/).

4. **Storage & Persistent Volumes**

   - [Longhorn](https://longhorn.io/) provides highly available storage within the cluster.
   - An NFS StorageClass is also available for NFS-backed persistent volumes.

5. **Home Network**

   - A Proxmox-hosted **pfSense** VM acts as firewall and DHCP/DNS server.
   - **Pi-hole** (LXC container) provides network-wide ad/tracker blocking and forwards DNS to pfSense Unbound.
   - **Tailscale** is used for secure remote access and subnet routing.
   - See [networking/README.md](networking/README.md) for full architecture details.

6. **Applications & Services**
   - **Blog** (Hugo-based container)
   - **Glance** (self-hosted dashboard / start page)
   - **Homepage** (a dynamic dashboard)
   - **Vaultwarden** (Bitwarden alternative)
   - **Immich** (photo management)
   - **Linkwarden** (bookmark manager)
   - **n8n** (automation workflow, including Renovate PR checks)
   - **PostgreSQL** clusters with [CloudNativePG](https://cloudnative-pg.io/)
   - **Monitoring** via Prometheus & Grafana
   - **Traefik** as the Ingress Controller
   - **External Secrets** for managing sensitive data
   - **Cert-Manager** for TLS certificates
   - **Descheduler** for rebalancing pods across nodes

---

## Folder Structure

```plaintext
.
├── ansible-k8s
│   ├── common/      # Common role tasks for all servers (installing curl, etc.)
│   ├── master/      # Master node role (installs K3s server, Tailscale, etc.)
│   ├── worker/      # Worker nodes role (K3s agents)
│   ├── longhorn/    # Role configuring node prerequisites for Longhorn
│   ├── playbook/    # All Ansible playbooks (K3s setup, Longhorn, maintenance, etc.)
│   └── inventory.ini # Ansible inventory
├── apps
│   ├── base/        # Base definitions for each application (Deployment, Service, Ingress)
│   └── the-big-ship/ # Environment-specific Kustomize overlays
├── clusters
│   └── the-big-ship/ # Flux Kustomizations for apps, databases, and infrastructure
├── database
│   ├── base/        # Base definitions for CloudNativePG clusters
│   └── the-big-ship/ # Overlays to deploy database instances for various apps
├── infrastructure
│   ├── base/        # HelmRelease definitions (cert-manager, traefik, longhorn, etc.)
│   └── the-big-ship/ # Overlays for production / main cluster environment
└── networking
    └── README.md    # Home network architecture (pfSense, Pi-hole, Proxmox, Tailscale)
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
- **playbook/**  
  Stores all the playbooks for provisioning and configuring the cluster, including:
  - `setup_k3s_cluster.yaml` — installs K3s on master and workers
  - `setup_longhorn.yaml` — configures Longhorn prerequisites cluster-wide
  - `get_kubeconfig.yaml`, `restart_k3s.yaml`, `shutdown_cluster.yaml`, `update_machines.yaml`, `prune-images.yaml`, `cleanup-k3s-sqlite.yaml` — maintenance playbooks
- **inventory.ini**  
  Specifies Ansible host groups for `master` and `workers`.

### apps

- **base/**  
  Kustomize bases for each application: Blog, External-Service, Glance, Homepage, Immich, Linkwarden, n8n, Vaultwarden.
- **the-big-ship/**  
  Overlays referencing `base` folders, often patching or adding configs (e.g., `homepage/configmap.yaml`) for the production environment.

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
  - **immich/** — PostgreSQL cluster for Immich
  - **linkwarden/** — PostgreSQL cluster for Linkwarden
  - **vaultwarden/** — PostgreSQL cluster for Vaultwarden
  - **rss-news/** — PostgreSQL cluster for an RSS news service
  - **playground/** — Example PostgreSQL cluster for testing purposes
- **the-big-ship/**  
  Environment overlays for each database cluster.

### infrastructure

- **base/**  
  HelmRelease definitions for core services, such as:
  - **cert-manager**
  - **cloudnative-pg**
  - **longhorn**
  - **monitoring** (Prometheus & Grafana)
  - **nfs-storageclass**
  - **traefik**
  - **external-secrets**
  - **descheduler**
- **the-big-ship/**  
  Kustomize overlays that adapt the base HelmRelease definitions to the environment, including a `certificate/` overlay for cluster-wide TLS certificates.

### networking

- **README.md**  
  Documents the home network architecture: Proxmox bridge setup, pfSense firewall and Unbound DNS, Pi-hole ad blocking, VLAN/subnet layout, firewall rules, and Tailscale integration. See [networking/README.md](networking/README.md).

---

## Key Technologies

- **K3s (Lightweight Kubernetes)**
- **Ansible** (Provisioning and Configuration Management)
- **Flux** (GitOps Continuous Delivery)
- **Kustomize** (Kubernetes Native Configuration Management)
- **Renovate** (Automated dependency update PRs)
- **Longhorn** (Container-Native Distributed Storage)
- **NFS StorageClass** (NFS-backed persistent volumes)
- **CloudNativePG** (PostgreSQL Operator for Kubernetes)
- **Traefik** (Ingress Controller)
- **Prometheus & Grafana** (Monitoring Stack)
- **External Secrets** (K8s Operator for managing secrets)
- **pfSense + Pi-hole** (Home network firewall and DNS/ad-blocking)
- **Tailscale** (Secure overlay network for remote access)

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

   - Ansible vault is used in `ansible-k8s/secrets.yaml`. Provide or update your own secrets (Tailscale auth key, etc.).

4. **Run Ansible Playbooks**

   - **Install K3s**:
     ```bash
     ansible-playbook ansible-k8s/playbook/setup_k3s_cluster.yaml -i ansible-k8s/inventory.ini
     ```
   - **Setup Longhorn** (once cluster is running):
     ```bash
     ansible-playbook ansible-k8s/playbook/setup_longhorn.yaml -i ansible-k8s/inventory.ini
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
- [x] **NFS StorageClass** for NFS-backed persistent volumes
- [x] **Monitoring stack** (Prometheus, Grafana) with CPU, memory, and uptime dashboards
- [x] **CloudNativePG** databases for Immich, Vaultwarden, Linkwarden, and RSS news
- [x] **Renovate** for automated dependency update PRs
- [x] **n8n automation** pipeline to check Renovate PRs for breaking changes
- [x] **Home network documentation** (pfSense, Pi-hole, Tailscale)

### Next Steps

- [ ] Move Vaultwarden back to Longhorn storage.
- [ ] Implement SQL database for Prometheus and Grafana.
- [ ] Implement GitHub Actions for linting, validation, and automated health checks.
- [ ] Implement alert rules for each monitored service.
- [ ] Integrate log aggregation with Loki.
- [ ] Implement backup and disaster recovery for each service.

---

## License

[MIT License](LICENSE) © 2025 [Wyson Cheng]

---

## Contact

**Maintainer**: [@ysonC](https://github.com/ysonC)  
If you have suggestions or questions, feel free to open an issue or submit a PR.

_Thank you for checking out my homelab setup!_
