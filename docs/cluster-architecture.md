# Cluster Architecture

## Purpose
This document describes the K3s cluster layout, how it is provisioned, and how workloads are managed.

## Cluster topology
- **K3s** runs on **3 Raspberry Pi 5 nodes**.
- The cluster is the primary runtime for most application workloads.

## Provisioning and bootstrap
- **Ansible** provisions and bootstraps the cluster.
- Provisioning is designed to be repeatable and version-controlled. See [Deployment and Operations Model](deployment-operations.md).

## Deployment model
- **FluxCD** reconciles the cluster from Git as the source of truth.
- **Helm/HelmRelease** is used for infrastructure components.
- **Kustomize/Kustomizations** organize application and service manifests.

## Workload placement
- Most services run in K3s.
- Selected infrastructure or appliance-like services remain outside Kubernetes for operational fit. See [Services Running](services-running.md).

## Persistence overview
- **Longhorn** provides cluster-native persistent volumes.
- **NFS** provides NAS-backed shared storage.
- Details are covered in [Storage Solution](storage-solution.md).
