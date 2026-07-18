# Proxmox Host Summary

## Purpose

This document summarizes the Proxmox virtualization hosts used in the homelab. These hosts provide the virtualization platform for future infrastructure services such as DNS, DHCP, monitoring, logging, backup, ticketing, and automation testing.

## Host Overview

### pve01

| Field | Value |
|---|---|
| Hostname | pve01 |
| Role | Main physical Proxmox host |
| Management IP | 10.10.0.10 |
| Reachability | Reachable |
| Proxmox version | 9.4.2 |
| CPU | 2x Intel Xeon Silver 4110 @ 2.10GHz |
| RAM | 32 GB |
| Storage | 250 GB SSD for root/thin-lvm, plus 2x 500 GB disks |
| Current VMs/containers | None yet |

Storage notes:

- 250 GB SSD:
  - root: approximately 65 GB
  - thin-lvm: approximately 152 GB
- 500 GB disk 1:
  - used as thin-lvm storage
- 500 GB disk 2:
  - used for ISO and other storage

### pve02

| Field | Value |
|---|---|
| Hostname | pve02 |
| Role | Secondary physical Proxmox host |
| Management IP | 10.10.0.11 |
| Reachability | Reachable |
| Proxmox version | 9.2.4 |
| CPU | Intel Core i7-2600 @ 3.40GHz |
| RAM | 12 GB |
| Storage | 120 GB SSD plus 1 TB disk |
| Current VMs/containers | None yet |

Storage notes:

- 120 GB SSD:
  - root: approximately 37 GB
  - thin-lvm: approximately 49 GB
- 1 TB disk:
  - available for additional VM, ISO, backup, or storage use

### pve03

| Field | Value |
|---|---|
| Hostname | pve03 |
| Role | Nested/testing Proxmox host |
| Management IP | 10.10.0.12 |
| Reachability | Reachable |
| Proxmox version | 9.4.2 |
| CPU | Intel Xeon E-2144G @ 3.60GHz, 3 cores assigned |
| RAM | 4 GB |
| Storage | 100 GB SSD-backed virtual disk |
| Current VMs/containers | None yet |

Nested host notes:

- pve03 runs as a nested Proxmox system inside Incus on ws01.
- pve03 is useful for testing Proxmox behavior, lab experiments, and safe learning.
- pve03 should not be treated as an independent physical failure domain.

## Current Platform State

At this stage, the Proxmox platform is reachable and documented, but no production homelab service VMs have been deployed yet.

Current state:

- pve01 is the main physical virtualization host.
- pve02 is a secondary physical host.
- pve03 is a nested testing host.
- No VMs or containers are running yet.
- Storage exists on each host but final workload placement is not yet defined.

## Network and Access Summary

All current Proxmox hosts are standalone hosts. They are not configured as a Proxmox cluster at this stage.

| Host | Cluster status | Network bridge | SSH access |
|---|---|---|---|
| pve01 | Standalone / not clustered | vmbr0 | SSH reachable using `sysadmin` user with key authentication; direct root SSH login disabled |
| pve02 | Standalone / not clustered | vmbr0 | SSH reachable using `sysadmin` user with key authentication; direct root SSH login disabled |
| pve03 | Standalone / not clustered | vmbr0 | SSH reachable using `sysadmin` user with key authentication; direct root SSH login disabled |

Access notes:

- Administrative SSH access uses a non-root `sysadmin` user.
- SSH key authentication is used for admin access.
- Direct root SSH login is disabled.
- This access model will be documented in more detail in the later identity and access control phase.

## Validation Performed

The following validation checks were performed during Phase 3.

### Reachability checks

| Check | Command or method | Expected result | Observed result |
|---|---|---|---|
| Ping pve01 | `ping -c 4 10.10.0.10` | Host replies | Passed |
| Ping pve02 | `ping -c 4 10.10.0.11` | Host replies | Passed |
| Ping pve03 | `ping -c 4 10.10.0.12` | Host replies | Passed |
| Proxmox web UI pve01 | Browser: `https://10.10.0.10:8006` | Login page reachable | Passed |
| Proxmox web UI pve02 | Browser: `https://10.10.0.11:8006` | Login page reachable | Passed |
| Proxmox web UI pve03 | Browser: `https://10.10.0.12:8006` | Login page reachable | Passed |

### Access checks

| Check | Command or method | Expected result | Observed result |
|---|---|---|---|
| SSH access pve01 | `ssh sysadmin@10.10.0.10` | Login succeeds with SSH key | Passed |
| SSH access pve02 | `ssh sysadmin@10.10.0.11` | Login succeeds with SSH key | Passed |
| SSH access pve03 | `ssh sysadmin@10.10.0.12` | Login succeeds with SSH key | Passed |
| Root SSH login | SSH configuration / login test | Direct root SSH login disabled | Passed |

### Proxmox host checks

| Check | Command | Expected result | Observed result |
|---|---|---|---|
| Proxmox version | `pveversion` | Version displayed | Passed |
| CPU information | `lscpu` | CPU model and core/thread information displayed | Passed |
| Memory information | `free -h` | RAM information displayed | Passed |
| Storage status | `pvesm status` | Proxmox storage listed | Passed |
| Block devices | `lsblk` | Local disks listed | Passed |
| Filesystem usage | `df -h` | Filesystem usage displayed | Passed |
| VM inventory | `qm list` | VM list displayed | Passed: no VMs currently exist |
| Container inventory | `pct list` | Container list displayed | Passed: no containers currently exist |
| Cluster status | `pvecm status` | Cluster or standalone status displayed | Passed: hosts are standalone/not clustered |
| Network bridge | `cat /etc/network/interfaces` | Bridge configuration displayed | Passed: `vmbr0` is used |


## Next Steps

Future phases will use this Proxmox platform to deploy infrastructure services in a controlled order:

1. Router/firewall foundation
2. IP plan, naming, and network zones
3. NTP/time service
4. DNS service
5. DHCP service
6. DHCP-DDNS and external DDNS
7. Reverse proxy and TLS
8. Monitoring, logging, backup, and ticketing services

