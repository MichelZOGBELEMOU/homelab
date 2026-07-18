# Proxmox VM and Container Inventory

## Purpose

This document tracks virtual machines and containers running on the homelab Proxmox platform.

At the start of Phase 3, no VMs or containers have been created yet. This is intentional. The platform is being documented before infrastructure services are deployed.

## Current VM Inventory

| VM ID | Name | Host | Role | IP Address | Status | Notes |
|---|---|---|---|---|---|---|
| N/A | N/A | N/A | N/A | N/A | Not created yet | No VMs exist at this stage |

## Current Container Inventory

| CT ID | Name | Host | Role | IP Address | Status | Notes |
|---|---|---|---|---|---|---|
| N/A | N/A | N/A | N/A | N/A | Not created yet | No containers exist at this stage |

## Planned Future Workloads

Future workloads may include:

| Planned Service | Possible Host | Purpose |
|---|---|---|
| DNS | pve01 or pve02 | Internal name resolution |
| DHCP | pve01 or router/firewall host depending on design | Controlled IP lease management |
| NTP/time service | pve01 or router/firewall host depending on design | Consistent time synchronization |
| Monitoring | pve01 or pve02 | System and service monitoring |
| Logging | pve01 or pve02 | Central log collection |
| Backup service | pve01, pve02, or dedicated storage target | Backup and restore validation |
| Zammad | pve01 or pve02 | Ticketing and incident/change tracking |

## Inventory Rules

Every future VM or container should be documented with:

- VM or container ID
- Name
- Host
- Role
- IP address
- CPU/RAM/storage allocation
- Network bridge
- Backup status
- Monitoring status
- Notes or risks

## Current Status

No VMs or containers are running yet.

This gives the homelab a clean starting point before deploying core infrastructure services.

