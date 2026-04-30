# Deployment and Operations Model

## Purpose
This document explains how the homelab is provisioned, deployed, and operated day to day.

## Provisioning
- **Ansible** is used to provision and bootstrap the K3s cluster.
- The goal is repeatable setup rather than manual configuration.

## GitOps deployment model
- **FluxCD** continuously reconciles cluster state from Git.
- **Git is the source of truth** for configuration and workloads.
- **Helm/HelmRelease** manages infrastructure components.
- **Kustomize/Kustomizations** organize application and service manifests.

## Operations and observability
- **Prometheus** and **Grafana** provide metrics and dashboards.
- **Uptime Kuma** monitors service availability from outside the cluster.

## Operational goals
- Reproducible, version-controlled infrastructure.
- Clear separation of responsibilities between cluster, infrastructure, and storage layers.

See [Cluster Architecture](cluster-architecture.md) for the runtime view and [Services Running](services-running.md) for placement context.
