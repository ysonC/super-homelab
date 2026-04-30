# Overall Architecture

## Purpose
This document provides a high-level view of the homelab design, the major platform layers, and the reasons for a hybrid architecture.

## Architecture at a glance
The homelab is a hybrid environment:
- **K3s** is the main application platform for most workloads.
- **Proxmox** provides virtualization and hosts core infrastructure services.
- **TrueNAS** on a ZimaBlade provides NAS-backed storage.
- **Longhorn** and **NFS** cover Kubernetes persistence needs.
- **Traefik** and **cert-manager** provide ingress and public TLS.
- **Tailscale** provides secure remote access.

This split is intentional to balance operational fit, resilience, and ease of management.

## Platform layers
- **Kubernetes compute layer:** K3s runs the majority of workloads. See [Cluster Architecture](cluster-architecture.md).
- **Virtualization and infrastructure layer:** Proxmox hosts pfSense, DNS, and other infrastructure services. See [Services Running](services-running.md).
- **NAS and storage layer:** TrueNAS on the ZimaBlade backs shared storage and long-term data. See [NAS and Storage Services](nas-storage-services.md).
- **Access and exposure layer:** Traefik, cert-manager, and Tailscale control service reachability. See [Ingress and Routing](ingress-routing.md) and [Networking](networking.md).

## Hardware footprint
- **3 Raspberry Pi 5 nodes** running K3s
- **1 N150 host** running Proxmox (including pfSense)
- **1 ZimaBlade** running TrueNAS

## Design rationale
- **Hybrid by design:** Some services remain outside Kubernetes for operational simplicity or because they are core infrastructure.
- **Avoid circular dependencies:** DNS (Pi-hole) stays outside the cluster so the cluster does not depend on itself for name resolution.
- **Infrastructure as code:** Provisioning and operations are declarative to keep the environment reproducible. See [Deployment and Operations Model](deployment-operations.md).
- **Security and controlled exposure:** Not all services are public; access is segmented by LAN, Tailscale, or Traefik with TLS.
