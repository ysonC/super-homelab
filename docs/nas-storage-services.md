# NAS and Storage Services

## Purpose
This document covers the NAS platform and storage-specific services that live outside Kubernetes.

## NAS platform
- The **ZimaBlade** hosts **TrueNAS**.
- The NAS layer is foundational to the homelab storage strategy.

## Core roles
- **NFS exports** feed shared storage into Kubernetes.
- **General file storage** and **long-term data retention** live on the NAS.
- **Media storage** is hosted here.
- **Jellyfin** runs via TrueNAS Docker for media services.

## Relationship to Kubernetes storage
Kubernetes workloads use Longhorn for cluster-native persistence and NFS for NAS-backed storage. See [Storage Solution](storage-solution.md).
