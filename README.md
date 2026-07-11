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
- See [`docs/home-network.md`](docs/home-network.md) for the detailed network design.

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
├── docs/                        # All supporting architecture and operations documentation
├── infrastructure/
│   ├── base/                    # Reusable platform manifests and HelmReleases
│   └── the-big-ship/            # Main environment infrastructure packages
├── templates/                   # Starter Kubernetes manifest template
└── renovate.json                # Automated dependency update configuration
```

## Workloads and Platform Services

### In-cluster applications

- **AI Agent View**
- **Blog** — Hugo-based site
- **Glance** — dashboard and start page
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
- [Detailed home-network architecture](docs/home-network.md)
- [Roadmap and backlog](docs/roadmap.md)

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

See [`docs/roadmap.md`](docs/roadmap.md) for the detailed backlog.

## Contact

**Maintainer:** [@ysonC](https://github.com/ysonC)

Suggestions and pull requests are welcome.
