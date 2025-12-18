# Repository Description

## Overview

**super-homelab** is a production-ready, cloud-native homelab infrastructure managed through GitOps principles. This repository demonstrates modern DevOps practices by combining infrastructure-as-code, declarative configuration management, and automated continuous delivery.

## Quick Summary

A personal homelab showcasing a complete Kubernetes-based infrastructure deployed on K3s, provisioned with Ansible, and managed through Flux CD with GitOps methodology. The setup includes self-hosted applications, monitoring stack, distributed storage, and automated certificate management.

## Core Purpose

- **Learning Platform**: Hands-on experience with cloud-native technologies and DevOps practices
- **Production-Ready Homelab**: Self-hosted services with high availability and automated management
- **GitOps Showcase**: Demonstrates declarative infrastructure and application deployment
- **DevOps Portfolio**: Real-world implementation of industry-standard tools and patterns

## Key Components

### Infrastructure Layer
- **K3s Cluster**: Lightweight Kubernetes distribution running on multi-node setup
- **Ansible Automation**: Automated provisioning and configuration of cluster nodes
- **Tailscale Network**: Secure overlay network for remote cluster access
- **Longhorn Storage**: Container-native distributed block storage

### Application Layer
- **Self-Hosted Services**: Blog, Homepage dashboard, Vaultwarden, Immich, n8n
- **Database Management**: PostgreSQL clusters via CloudNativePG operator
- **Monitoring Stack**: Prometheus and Grafana for metrics and visualization
- **Automation Platform**: n8n workflow automation engine

### Platform Services
- **Traefik**: Ingress controller and reverse proxy
- **Cert-Manager**: Automated TLS certificate management
- **External Secrets**: Kubernetes secrets management operator
- **Flux CD**: GitOps continuous delivery operator
- **Descheduler**: Pod rescheduling for optimal resource distribution

## Technology Stack

| Category | Technologies |
|----------|-------------|
| **Orchestration** | Kubernetes (K3s) |
| **Provisioning** | Ansible |
| **GitOps** | Flux CD, Kustomize |
| **Storage** | Longhorn, NFS |
| **Networking** | Tailscale, Traefik |
| **Database** | PostgreSQL (CloudNativePG) |
| **Monitoring** | Prometheus, Grafana |
| **Security** | Cert-Manager, External Secrets |
| **Dependencies** | Renovate (automated updates) |

## Architecture Highlights

1. **Declarative Configuration**: All infrastructure and applications defined as code
2. **GitOps Workflow**: Git as single source of truth, automated reconciliation
3. **High Availability**: Distributed storage, multiple worker nodes, automated failover
4. **Security First**: TLS everywhere, secrets management, secure overlay networking
5. **Automated Updates**: Renovate bot for dependency management and security patches
6. **Observability**: Comprehensive monitoring and metrics collection

## Repository Structure

```
super-homelab/
├── ansible-k8s/           # Ansible roles and playbooks for cluster provisioning
├── apps/                  # Application definitions (base + overlays)
├── clusters/              # Flux Kustomizations for GitOps
├── database/              # Database cluster definitions
├── infrastructure/        # Core infrastructure HelmReleases
├── networking/            # Network configurations
└── templates/             # Reusable templates
```

## Target Audience

- DevOps Engineers learning Kubernetes and GitOps
- System Administrators building self-hosted infrastructure
- Cloud Engineers exploring cloud-native technologies
- Homelab Enthusiasts seeking production-grade setups

## Current Status

✅ **Operational**: Core infrastructure and services deployed and running
🚧 **In Development**: CI/CD pipelines, advanced security scanning, expanded monitoring

## Links

- **Repository**: [github.com/ysonC/super-homelab](https://github.com/ysonC/super-homelab)
- **Maintainer**: [@ysonC](https://github.com/ysonC)
- **License**: MIT

---

*For detailed installation instructions, architecture diagrams, and complete documentation, see [README.md](README.md)*
