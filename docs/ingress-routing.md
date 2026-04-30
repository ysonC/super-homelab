# Ingress and Routing

## Purpose
This document describes how services are exposed and how inbound traffic is routed.

## Ingress layer
- **Traefik** is the main ingress controller and reverse proxy for Kubernetes.
- It provides hostname-based routing and TLS termination for in-cluster services.

## TLS certificates
- **cert-manager** automates certificate management.
- **Let’s Encrypt** is the primary public certificate authority.

## Exposure strategy
- Services are exposed based on need: **public**, **LAN-only**, or **Tailscale-only**.
- Traefik may also front selected non-Kubernetes services when appropriate.

For network reachability and access paths, see [Networking](networking.md).
