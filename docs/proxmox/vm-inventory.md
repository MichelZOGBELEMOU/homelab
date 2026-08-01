# Proxmox VM and Container Inventory

## Purpose

This document tracks virtual machines and containers running on the homelab Proxmox platform.

The goal is to document where each workload runs, what role it has, which IP address it uses, and whether it has been validated.

## Current VM Inventory

| VM ID | Name | Host | Role | IP Address | VLAN | Status | Notes |
|---|---|---|---|---|---:|---|---|
| Needs verification | dns01 | pve02 | Primary internal DNS server | 10.10.20.10 | 20 | BIND9 installed / default config validated | Debian 13 installed. |
| Not created yet | dns02 | pve03 | Secondary internal DNS server | 10.10.20.11 | 20 | Planned | Build after dns01 is installed, configured, and validated. |

## Current Container Inventory

| CT ID | Name | Host | Role | IP Address | VLAN | Status | Notes |
|---|---|---|---|---|---:|---|---|
| N/A | N/A | N/A | N/A | N/A | N/A | Not created yet | No containers are documented at this stage. |

## dns01 Base Install Validation

| Item | Value |
|---|---|
| Hostname | dns01 |
| Proxmox host | pve02 |
| Operating system | Debian 13 |
| Role | Primary internal DNS server |
| IP address | 10.10.20.10/24 |
| Gateway | 10.10.20.1 |
| VLAN | VLAN 20 / SRV |
| SSH from admin machine | Passed |
| Gateway ping | Passed |
| Internet IP ping | Passed |
| External DNS resolution | Passed |
| Current status | Ready for BIND9 installation |

## dns01 Partition Layout

| Mount or Volume | Size | Type | Purpose |
|---|---:|---|---|
| /boot | 1 GB | ext4 | Boot files |
| / | 10 GB | LVM | Root filesystem |
| swap | 2 GB | LVM | Swap space |
| /var | 7 GB | LVM | Logs, package cache, and service data |

Operational note:

- A separate `/var` partition is useful for service logs and package/cache data.
- The `/var` size should be monitored after BIND9 is installed because DNS logs and package cache can grow over time.

## Planned Future Workloads

| Planned Service | Planned Host | Purpose | Status |
|---|---|---|---|
| dns01 | pve02 | Primary internal DNS service | Base OS installed |
| dns02 | pve03 | Secondary DNS and zone-transfer practice | Planned |
| dhcp01 | To be decided | Future DHCP service if moved from VyOS | Deferred |
| ntp01 | To be decided | Future NTP/time service | Deferred |
| monitor01 | To be decided | Future monitoring service | Deferred |
| log01 | To be decided | Future logging service | Deferred |
| backup01 | To be decided | Future backup service | Deferred |
| zammad01 | To be decided | Future ticketing/service desk | Deferred |

## Inventory Rules

Every future VM or container should be documented with:

- VM or container ID
- Name
- Proxmox host
- Role
- IP address
- VLAN
- CPU allocation
- RAM allocation
- Disk allocation
- Network bridge
- Backup status
- Monitoring status
- Validation status
- Notes or risks

## Current Status

The first service VM, `dns01`, has been installed and validated at the base OS level.

BIND9 has not been installed yet.

The next step is to install and configure BIND9 on `dns01`.

