# Known Unknowns and Risks

## Purpose

This document tracks current unknowns, risks, and follow-up items discovered during the homelab inventory phase.

The goal is to avoid guessing. Unknowns are recorded clearly so they can be verified or improved later in the correct phase.

## Principles

- Unknowns are not failures.
- A documented unknown is better than a guessed fact.
- Risks should be recorded before they are fixed.
- Follow-up work should be handled in the correct phase, not all at once.

## Network and firewall unknowns

### Firewall policy

- Area: router/firewall
- Related device: ZBOX-CI327NANO-GS-01 / VyOS
- Current status: firewall rules are not fully documented yet
- Risk:
  - The security posture of the homelab is not fully known.
- Later phase:
  - Phase 4 — Router/firewall foundation
  - Phase 17 — Security baseline

### VLAN and network segmentation

- Area: switching/network design
- Related device: TP-Link TL-SG108E
- Current status: VLANs are not active yet
- Risk:
  - The current network is flat.
  - Lab, admin, management, and client traffic are not separated.
- Later phase:
  - Phase 5 — IP plan, naming, and network zones

### Formal IP plan

- Area: IP addressing
- Related devices:
  - VyOS router/firewall
  - TP-Link switch
  - Proxmox hosts
  - admin/control nodes
  - ipTIME AX2004T
- Current status:
  - Current IP addresses are documented in `ip-inventory.md`.
  - A formal long-term IP plan is not documented yet.
- Risk:
  - Future services may receive inconsistent addresses or naming.
- Later phase:
  - Phase 5 — IP plan, naming, and network zones

### ipTIME management address

- Area: wireless/AP management
- Related device: ipTIME AX2004T
- Current status:
  - Device is in AP mode.
  - DHCP is disabled.
  - Management IP is 192.168.0.1/24.
- Risk:
  - Management IP is outside the main 10.10.0.0/24 LAN.
  - Long-term management access plan is not documented.
- Later phase:
  - Phase 5 — IP plan, naming, and network zones

## Server and virtualization unknowns

### Proxmox host responsibilities

- Area: virtualization operations
- Related devices:
  - pve01
  - pve02
  - pve03
- Current status:
  - pve01 is the main Proxmox host.
  - pve02 is the secondary Proxmox host.
  - pve03 is a nested lab/testing Proxmox VM.
- Risk:
  - Workload placement, backup responsibility, and recovery expectations are not documented yet.
- Later phase:
  - Phase 3 — Proxmox virtualization platform
  - Phase 14 — Backup and restore
  - Phase 15 — Monitoring and alerting

### Nested Proxmox failure domain

- Area: virtualization design
- Related device: nested Proxmox / pve03
- Current status:
  - pve03 is a nested Proxmox VM hosted through HP Z2 / ws01.
- Risk:
  - It may be misunderstood as an independent physical host.
- Later phase:
  - Phase 3 — Proxmox virtualization platform
- Note:
  - pve03 should be documented as a lab/testing node, not an independent physical failure domain.

## Management access unknowns

### Admin/control node access

- Area: administration
- Related devices:
  - HP Z2 / ws01
  - LG Gram / lg-gram
- Current status:
  - Local login exists.
  - SSH status is not fully documented.
- Risk:
  - Admin access paths and security baseline are unclear.
- Later phase:
  - Phase 12 — Identity and access control
  - Phase 17 — Security baseline

### HPE iLO access policy

- Area: out-of-band management
- Related device: HPE iLO 5
- Current status:
  - iLO Web UI is reachable.
  - Access policy is not documented yet.
- Risk:
  - iLO can control physical server power and remote console access.
  - It must be protected carefully.
- Later phase:
  - Phase 12 — Identity and access control
  - Phase 17 — Security baseline

## Operations unknowns

### Backup and recovery

- Area: operations/reliability
- Related devices:
  - pve01
  - pve02
  - pve03
  - admin/control nodes
- Current status: backup and restore responsibilities are not documented yet
- Risk:
  - A failure could cause data or VM loss if recovery is not planned.
- Later phase:
- Phase 14 — Backup and restore

### Monitoring and alerting

- Area: operations/reliability
- Related devices:
  - router/firewall
  - switch
  - Proxmox hosts
  - admin/control nodes
- Current status: monitoring and alerting are not configured/documented yet
- Risk:
  - Failures may not be noticed quickly.
- Later phase:
  - Phase 15 — Monitoring and alerting

### Logging

- Area: troubleshooting
- Related devices:
  - router/firewall
  - Proxmox hosts
  - admin/control nodes
- Current status: centralized logging is not configured/documented yet
- Risk:
  - Troubleshooting may depend on scattered local logs.
- Later phase:
  - Phase 16 — Central logging

## Summary

The main current unknowns and risks are:

- firewall policy
- VLAN/network segmentation
- formal IP plan
- ipTIME long-term management addressing
- Proxmox workload and recovery responsibilities
- admin access/security baseline
- iLO access policy
- backup and restore responsibilities
- monitoring and alerting
- centralized logging

These items should not all be fixed during the inventory phase. They should be handled gradually in the correct later phases.


