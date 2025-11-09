# Home Network Architecture: pfSense, Pi-hole, Proxmox, and Tailscale

## Overview

This document outlines the network design and DNS resolution flow in a homelab environment built on top of **Proxmox VE**, with virtualized **pfSense**, **Pi-hole (LXC)**, and integration with **Tailscale** for secure remote access.

---

## Infrastructure Summary

### Virtualization Host: Proxmox VE

- **Platform**: Proxmox Virtual Environment (Proxmox VE)
- **Network Bridges**:
  - `vmbr0`: Bridge for WAN (internet-facing interface)
  - `vmbr1`: Bridge for LAN (internal network)
  - `vmbr2`: Bridge for OPT1 (segregated or isolated network)

### Virtual Machines and Containers

#### pfSense (Virtual Machine)

- **Primary Functions**:
  - Acts as the network firewall
  - Provides DHCP services for internal networks
  - Functions as the local DNS resolver via Unbound
- **Assigned Interfaces**:
  - `net0` connected to `vmbr0` (WAN)
  - `net1` connected to `vmbr1` (LAN)
  - `net2` connected to `vmbr2` (OPT1)

#### Pi-hole (LXC Container)

- **Primary Functions**:
  - Network-wide DNS sinkhole for ad and tracker blocking
  - Forwards DNS requests to pfSense's Unbound resolver (port 53)
- **Placement**:
  - Hosted in a lightweight LXC container for efficiency

#### Other Guests

- Additional Proxmox guests for development or utility purposes (e.g., Linux, Rust, test environments)

---

## Network Interfaces

| Interface | Role                  | Subnet          | Description                                               |
| --------- | --------------------- | --------------- | --------------------------------------------------------- |
| `LAN`     | Primary wired network | `10.8.8.0/24`   | Contains pfSense, Pi-hole, homelab enviroment and main PC |
| `OPT1`    | Wi-Fi/IoT VLAN        | `10.10.10.0/24` | Wireless and segregated devices                           |
| `WAN`     | Internet uplink       | ISP-assigned    | Connected to modem/gateway                                |

---

## Firewall Rules Overview (pfSense)

This document outlines the active firewall rules configured on the pfSense instance running inside a Proxmox environment.

---

### WAN Rules

| Protocol | Source            | Source Port | Destination | Destination Port | Description            |
| -------- | ----------------- | ----------- | ----------- | ---------------- | ---------------------- |
| \*       | RFC 1918 networks | \*          | \*          | \*               | Block private networks |
| \*       | Reserved (IANA)   | \*          | \*          | \*               | Block bogon networks   |

---

### LAN Rules

| Protocol | Source      | Source Port | Destination | Destination Port | Gateway | Description                   |
| -------- | ----------- | ----------- | ----------- | ---------------- | ------- | ----------------------------- |
| \*       | \*          | \*          | LAN Address | 80               | \*      | Anti-Lockout Rule             |
| IPv4 \*  | LAN subnets | \*          | \*          | \*               | none    | Default allow LAN to any rule |
| IPv6 \*  | LAN subnets | \*          | \*          | \*               | none    | Default allow LAN IPv6 to any |

---

## OPT1 Rules

| Protocol     | Source       | Source Port | Destination | Destination Port | Gateway | Description                 |
| ------------ | ------------ | ----------- | ----------- | ---------------- | ------- | --------------------------- |
| IPv4 TCP/UDP | OPT1 subnets | \*          | 10.8.8.8    | 53 (DNS)         | none    | Allow DNS to Pi-hole        |
| IPv4 \*      | OPT1 subnets | \*          | ! RFC1918   | \*               | none    | Allow OPT1 to Internet only |
| IPv6 \*      | OPT1 subnets | \*          | \*          | \*               | none    | (rule present but disabled) |
| IPv4 \*      | OPT1 subnets | \*          | \*          | \*               | none    | (rule present but disabled) |

---

**Note:** This setup assumes DNS resolution is handled by the Pi-hole instance at `10.8.8.8`, with additional internet access granted only to external (non-RFC1918) IPs from OPT1. LAN is broadly permitted to communicate internally and externally, and standard WAN protection is in place to block private and bogon networks.

---

## DNS Resolution Architecture

### Design Objective

All clients resolve DNS via Pi-hole, which then forwards queries to pfSense for upstream resolution using Unbound.

### Resolution Flow

Client → Pi-hole (10.8.8.8) → pfSense (10.8.8.1 Unbound) → Root Servers / Upstream DNS

### DNS Roles

- **Pi-hole (10.8.8.8)**:
  - Receives all DNS queries from LAN and OPT1 clients
  - Performs domain blocking and logging
  - Forwards all upstream queries to pfSense at `10.8.8.1`

- **pfSense (10.8.8.1)**:
  - Runs Unbound DNS Resolver
  - Performs full recursive DNS resolution
  - Acts as upstream DNS for Pi-hole

---

## DHCP and DNS Configuration

### pfSense DHCP Server (LAN and OPT1)

- **DNS server advertised to clients**: `10.8.8.8` (Pi-hole)
- Clients are assigned IPs via DHCP from pfSense
- No DNS fallback to pfSense or external resolvers is provided to clients directly

### Pi-hole DNS Settings

- **Upstream DNS**: `10.8.8.1#53`
- **Blocking lists**: Custom + default gravity lists
- **Interface**: Bound to LAN
- **Conditional Forwarding**: Disabled
- **DoH/DoT**: Not used

---

## Tailscale Integration

### Role

- Used to enable secure remote access to LAN/OPT1 devices
- Tailscale installed on selected endpoints and on pfSense

### Subnet Routing

- Tailscale advertised `10.8.8.0/24` as a subnet route
  - For client on OPT1, such as my phone, to access my homelab services.

#### Resulted Issue

- Resulted in connected clients routing traffic via pfSense over Tailscale
- Caused Pi-hole to see DNS queries as originating from pfSense (10.8.8.1) instead of actual client IPs

**Resolution**

- Disabled subnet route advertisements to LAN clients
- Used `tailscale up --accept-routes=faluse --accept-dns=false` on local machines to bypass Tailscale DNS hijacking
- After change, Pi-hole correctly logs per-client IPs again

---
