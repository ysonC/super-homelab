# Homelab

This repository contains the infrastructure-as-code and GitOps configuration for my personal homelab. **Ansible** provisions a highly available K3s cluster, while **FluxCD**, **Kustomize**, and **HelmRelease** resources continuously reconcile applications, databases, and platform services from Git.

> This is a personal environment rather than a turnkey distribution. The manifests document the design and may require changes to hostnames, addresses, storage, domains, and secrets before reuse.

## Architecture Overview

![Homelab Architecture Diagram](https://github.com/user-attachments/assets/acfd7ea4-0e29-4214-9965-78da89e783d5)

### Compute and control plane

- K3s runs on three Raspberry Pi 5 nodes: `ahoy1`, `ahoy2`, and `ahoy3`.
- All three nodes are K3s server/control-plane nodes and form a highly available **embedded-etcd** cluster.
- Every Pi boots and runs its workloads from SSD storage rather than microSD.
- Ansible installs K3s, initializes embedded etcd on the first server, and joins the remaining servers to the control plane.

### GitOps

- Flux watches the `main` branch of this repository.
- The cluster entry point is [`clusters/the-big-ship/`](clusters/the-big-ship/).
- Flux reconciles infrastructure first; application and database Kustomizations depend on it.
- Each immediate child under `apps/the-big-ship`, `database/the-big-ship`, and `infrastructure/the-big-ship` is a standalone Kustomize package. The environment roots do not contain aggregate `kustomization.yaml` files.

### Storage

- [Longhorn](https://longhorn.io/) provides distributed, cluster-native persistent volumes across the SSD-backed nodes.
- An NFS StorageClass provides NAS-backed shared storage from TrueNAS.
- CloudNativePG manages PostgreSQL workloads and their persistent storage.

### Network and service exposure

- A Proxmox-hosted **pfSense** VM provides firewall, routing, DHCP, and Unbound DNS.
- **Pi-hole** runs in an LXC container and provides network-wide DNS filtering.
- **Traefik** handles Kubernetes ingress and routes selected external services.
- **cert-manager** provisions TLS certificates using the Cloudflare DNS challenge.
- **Cloudflare Tunnel** and **Tailscale** provide controlled public and private access paths.
- See [`networking/README.md`](networking/README.md) for the detailed network design.

## Repository Structure

```text
.
├── ansible-k8s/                 # K3s provisioning and node maintenance
│   ├── common/                  # Tasks shared by all cluster nodes
│   ├── master/                  # K3s server and embedded-etcd setup
│   ├── longhorn/                # Longhorn host prerequisites
│   ├── playbook/                # Provisioning and operational playbooks
│   ├── group_vars/              # Shared Ansible variables
│   └── inventory.ini            # Control-plane inventory
├── apps/
│   ├── base/                    # Reusable application manifests
│   └── the-big-ship/            # Main environment application packages
├── clusters/
│   └── the-big-ship/            # Flux bootstrap and reconciliation graph
├── database/
│   ├── base/                    # Database and data-service manifests
│   └── the-big-ship/            # Main environment database packages
├── docs/                        # Architecture and operations documentation
├── infrastructure/
│   ├── base/                    # Reusable platform manifests and HelmReleases
│   └── the-big-ship/            # Main environment infrastructure packages
├── networking/                  # Detailed home-network documentation
├── templates/                   # Starter Kubernetes manifest template
├── renovate.json                # Automated dependency update configuration
└── Todo.md                      # Detailed backlog
```

## Workloads and Platform Services

### In-cluster applications

- **AI Agent View**
- **Blog** — Hugo-based site
- **Glance** — dashboard and start page
- **Homepage** — service dashboard
- **Immich** — photo management
- **n8n** — workflow automation, including Renovate PR checks
- **Vaultwarden** — Bitwarden-compatible password manager

The repository also contains test packages for generic workloads and Vaultwarden. These are kept separate from the main application list because their presence in Git does not necessarily mean they are production services.

### External-service routes

Traefik routing manifests expose selected services that run outside Kubernetes, including:

- Hermes
- Jellyfin
- NAS / TrueNAS
- OpenClaw
- Pi-hole
- Proxmox
- Uptime Kuma

### Databases and data services

- **CouchDB**
- **CloudNativePG PostgreSQL clusters** for Immich, Linkwarden, and RSS news
- **Playground PostgreSQL cluster** for testing

### Cluster infrastructure

- **cert-manager** and service certificates
- **Cloudflare Tunnel** (`cloudflared`)
- **CloudNativePG operator**
- **Descheduler**
- **Dozzle**
- **External Secrets Operator**
- **Longhorn**
- **NFS StorageClass**
- **Prometheus and Grafana**
- **Traefik**

For a placement-oriented overview, see [`docs/services-running.md`](docs/services-running.md).

## Provisioning and Bootstrap

### Prerequisites

- Three compatible Linux hosts with SSD storage
- Passwordless SSH access from the Ansible controller
- Ansible
- `kubectl`
- Flux CLI and GitHub CLI for a fresh GitOps bootstrap
- Environment-specific domains, addresses, and secret values

### 1. Clone the repository

```bash
git clone https://github.com/ysonC/super-homelab.git
cd super-homelab
```

### 2. Configure Ansible

Update [`ansible-k8s/inventory.ini`](ansible-k8s/inventory.ini) with the three K3s server addresses and SSH users. Review variables under `ansible-k8s/group_vars/` and provide the required encrypted Ansible Vault values in `ansible-k8s/secrets.yaml`.

Do not commit unencrypted credentials or reuse the repository's environment-specific secret references without reviewing them.

### 3. Provision K3s and storage prerequisites

```bash
ansible-playbook \
  -i ansible-k8s/inventory.ini \
  ansible-k8s/playbook/setup_k3s_cluster.yaml

ansible-playbook \
  -i ansible-k8s/inventory.ini \
  ansible-k8s/playbook/setup_longhorn.yaml
```

The K3s playbook initializes the first inventory host with `--cluster-init`, then joins the other hosts as K3s servers in the same embedded-etcd cluster.

### 4. Bootstrap Flux

The committed Flux sync configuration points to:

```text
https://github.com/ysonC/super-homelab.git
path: clusters/the-big-ship
branch: main
```

For a new fork or replacement cluster, authenticate the Flux and GitHub CLIs, then bootstrap using your own owner and repository:

```bash
flux check --pre

flux bootstrap github \
  --owner=<github-owner> \
  --repository=<repository-name> \
  --branch=main \
  --path=clusters/the-big-ship \
  --personal
```

Bootstrap writes or updates the generated files under `clusters/the-big-ship/flux-system/`. Commit those changes to the repository Flux will monitor. Do not write generated manifests to a top-level `flux-system/` directory.

### 5. Validate changes locally

Each environment child is independently buildable. For example:

```bash
kubectl kustomize apps/the-big-ship/n8n >/dev/null
kubectl kustomize database/the-big-ship/immich >/dev/null
kubectl kustomize infrastructure/the-big-ship/traefik >/dev/null
```

After changes are merged, Flux reconciles the cluster from Git. Prefer Git changes over manual edits to Flux-managed resources.

## Key Technologies

- **K3s** with embedded etcd
- **Ansible**
- **FluxCD**
- **Kustomize**
- **Helm / HelmRelease**
- **Renovate**
- **Longhorn** and NFS
- **CloudNativePG**
- **Traefik** and Cloudflare Tunnel
- **cert-manager**
- **External Secrets Operator**
- **Prometheus and Grafana**
- **Proxmox, pfSense, Pi-hole, TrueNAS, and Tailscale**

## Documentation

- [Documentation index](docs/README.md)
- [Overall architecture](docs/architecture-overview.md)
- [Cluster architecture](docs/cluster-architecture.md)
- [Deployment and operations](docs/deployment-operations.md)
- [Ingress and routing](docs/ingress-routing.md)
- [Storage solution](docs/storage-solution.md)
- [Services running](docs/services-running.md)
- [Detailed home-network architecture](networking/README.md)

## Current Status and Roadmap

### Implemented

- [x] Three-node K3s control plane with embedded etcd
- [x] SSD-backed storage on every Raspberry Pi node
- [x] Ansible-based K3s provisioning and maintenance
- [x] Flux GitOps reconciliation
- [x] Traefik ingress and cert-manager TLS using Cloudflare DNS challenge
- [x] Cloudflare Tunnel and Tailscale access paths
- [x] Longhorn and NFS storage classes
- [x] Vaultwarden on Longhorn storage
- [x] Prometheus and Grafana monitoring
- [x] CloudNativePG-managed PostgreSQL workloads
- [x] Renovate dependency updates
- [x] n8n automation for reviewing Renovate changes

### Next steps

- [ ] Add GitHub Actions for linting, manifest validation, and basic health checks
- [ ] Add service-specific alert rules
- [ ] Integrate Loki for log aggregation
- [ ] Define and test backup and disaster-recovery procedures for each stateful service

See [`Todo.md`](Todo.md) for the detailed backlog.

## Contact

**Maintainer:** [@ysonC](https://github.com/ysonC)

Suggestions and pull requests are welcome.
