# Services Running

## Purpose
This document summarizes known services and explains why some live inside Kubernetes while others remain outside.

## In-cluster infrastructure services
These are the confirmed platform services running in K3s:
- **Traefik** (ingress controller)
- **cert-manager** (public TLS automation)
- **FluxCD** (GitOps reconciliation)
- **Longhorn** (persistent storage)
- **CloudNativePG** (PostgreSQL operator)
- **external-secrets** (secret sync)
- **Prometheus** (metrics collection)
- **Grafana** (dashboards and visualization)
- **descheduler** (periodic workload rebalancing)

## In-cluster application workloads
The repo currently contains application manifests or overlays for:
- **Blog**
- **Glance**
- **Homepage**
- **Immich**
- **Linkwarden**
- **n8n**
- **Vaultwarden**
- **Vaultwarden testing**
- **test** manifests
- **external-service** routes for selected services that live outside Kubernetes

## Services outside Kubernetes
These services run outside the cluster for operational or architectural reasons and are represented in the repo where ingress or documentation is needed:
- **pfSense** on Proxmox (network edge)
- **Pi-hole** in Proxmox LXC (DNS)
- **Uptime Kuma** in Proxmox LXC (service monitoring)
- **OpenClaw** in Proxmox LXC
- **TrueNAS** on the ZimaBlade (NAS platform)
- **Jellyfin** via TrueNAS Docker (media)
- **Proxmox** on the N150 host (virtualization layer)

## Placement rationale
- Core network services stay outside Kubernetes to avoid circular dependencies.
- Appliance-like services are kept on Proxmox or TrueNAS when that is a better operational fit.

For networking and access details, see [Networking](networking.md) and [Ingress and Routing](ingress-routing.md).
