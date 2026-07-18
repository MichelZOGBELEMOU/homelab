# Hardware Risk Notes

## Purpose

This document records hardware limitations, risks, and known constraints for the homelab virtualization platform.

The goal is not to make the lab perfect. The goal is to document the environment honestly and understand the operational risks.

## Hardware Summary

### pve01

pve01 is the main physical Proxmox host.

Known facts:

- Hardware: HPE DL360 Gen10
- CPU: 2x Intel Xeon Silver 4110
- RAM: 32 GB
- Storage:
  - 250 GB SSD
  - 2x 500 GB disks
- Role: Main physical virtualization host

Operational notes:

- This is the strongest current Proxmox host.
- It should host the most important infrastructure service VMs first.
- Storage planning should be completed before deploying many workloads.

### pve02

pve02 is the secondary physical Proxmox host.

Known facts:

- Hardware: Samsung desktop
- CPU: Intel Core i7-2600
- RAM: 12 GB
- Storage:
  - 120 GB SSD
  - 1 TB disk
- Role: Secondary virtualization host

Operational notes:

- Lower RAM than pve01.
- Useful for secondary services, testing, and non-critical workloads.
- Workload placement should consider age and resource limits.

### pve03

pve03 is a nested Proxmox host.

Known facts:

- Runs inside Incus on ws01
- CPU: Intel Xeon E-2144G, 3 cores assigned
- RAM: 4 GB
- Storage: 100 GB virtual disk
- Role: Nested/testing Proxmox host

Operational notes:

- pve03 is useful for testing and learning.
- pve03 is not an independent physical failure domain.
- If ws01 or the Incus layer fails, pve03 is affected.
- pve03 should not host important services that are meant to prove physical redundancy.

## Current Risks

| Risk | Impact | Mitigation |
|---|---|---|
| No VMs or services deployed yet | Platform is documented but not yet operationally useful | Deploy services in later phases after network and naming plans |
| Storage layout not fully standardized | Future VM placement may become inconsistent | Document storage purpose before creating many VMs |
| pve03 is nested | Not suitable as independent redundancy | Use pve03 only for testing/lab experiments |
| Backup status not yet defined | VM recovery is not proven | Address backup and restore in a later backup phase |
| Monitoring not yet deployed | Host/service failures may not be visible automatically | Add monitoring in the monitoring phase |
| Hosts are standalone and not clustered | No Proxmox HA or live migration is available at this stage | Treat hosts as independent standalone systems until clustering is intentionally designed |


## Current Unknowns

The following items should be confirmed later:


- Final storage purpose for each disk
- Backup target and backup schedule
- Which host will run each infrastructure service
- Whether any shared storage will be introduced later

## Operational Principle

A homelab does not need enterprise-grade hardware to be useful as a portfolio.

However, production-style operations require that hardware roles, limits, risks, and recovery assumptions are documented clearly.

