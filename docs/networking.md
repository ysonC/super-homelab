# Networking

## Purpose
This document explains how connectivity, DNS, and remote access are structured across the homelab.

## Core network components
- **pfSense** runs on Proxmox and provides internet connectivity for the entire homelab.
- **Pi-hole** runs in a Proxmox LXC and provides DNS services.
- **Tailscale** provides secure remote access and private overlay networking.

## Access paths
Services are reachable through different paths depending on their exposure:
- **Internal LAN access** for local services.
- **Tailscale private access** for secure remote administration.
- **Public domain-based access** via Traefik for selected services. See [Ingress and Routing](ingress-routing.md).

## Design notes
- DNS is intentionally outside Kubernetes to avoid circular dependency problems.
- Proxmox is a critical infrastructure host because it runs pfSense.
- Not all services are public; some are LAN-only or Tailscale-only by design.
