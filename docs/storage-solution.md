# Storage Solution

## Purpose
This document describes how persistent storage is provided to Kubernetes workloads and how it connects to the NAS layer.

## Storage layers
- **Longhorn** provides Kubernetes-native persistent volumes.
- **NFS** provides shared storage backed by the NAS.

## Backing storage
The storage backend is the **TrueNAS** system running on the ZimaBlade, which provides:
- NAS-backed storage for Kubernetes
- general file and media storage
- long-term data outside the cluster

## Rationale
- **Longhorn** fits cluster-native persistence needs.
- **NFS** supports shared storage and NAS-backed workloads.

For NAS platform details, see [NAS and Storage Services](nas-storage-services.md).
