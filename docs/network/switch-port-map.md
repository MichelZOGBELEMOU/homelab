# Switch Port Map

## Purpose

This document records the planned TP-Link TL-SG108E port usage and VLAN behavior.

## Current Switch

- Device: TP-Link TL-SG108E
- Management IP: `10.10.0.100`
- Current VLAN status: not active yet

## Current Physical Connections

| Port | Connected Device | Current Role |
|---:|---|---|
| 1 | VyOS `eth2` | Router/firewall LAN uplink |
| 2 | HPE iLO | Out-of-band management |
| 3 | pve01 | Main Proxmox host |
| 4 | ws01 | Admin workstation / nested pve03 host |
| 5 | ipTIME AX2004T | Wi-Fi access point |
| 6 | pve02 | Secondary Proxmox host |
| 7 | Spare | Unused |
| 8 | Spare / emergency | Recovery access port |

## Planned Port Roles

| Port | Planned Mode | VLANs | Notes |
|---:|---|---|---|
| 1 | Trunk | 10,20,30,40,50,60 | Uplink to VyOS |
| 2 | Access | 10 | HPE iLO management |
| 3 | Trunk | 10,20,60 | pve01 management and future service/DMZ VMs |
| 4 | Trunk | 10,30,60 | ws01 admin workstation, nested pve03, and admin/DMZ testing |
| 5 | Access | 30 | ipTIME AP does not support VLANs; used for Admin Wi-Fi / lg-gram |
| 6 | Trunk or access first | 10,20,60 | pve02, depending on Proxmox VLAN readiness |
| 7 | Spare | TBD | Reserved |
| 8 | Emergency access | Legacy or Management | Keep available for recovery |

## ipTIME AP VLAN Limitation

The ipTIME AX2004T does not support VLAN tagging or multiple VLAN SSIDs.

Because of this, the switch port connected to the ipTIME AP must be a single access VLAN.

Current decision:

- TP-Link port 5: access VLAN 30
- AP purpose: Admin Wi-Fi
- Main Wi-Fi admin device: `lg-gram`

This means the AP should not be used for guest or untrusted devices unless it is later moved to a different VLAN.


## Notes

- Port 1 must become a trunk only after VyOS VLAN interfaces are ready.
- Port 8 should remain available as an emergency access port.
- Proxmox trunk ports require VLAN-aware bridge configuration.

