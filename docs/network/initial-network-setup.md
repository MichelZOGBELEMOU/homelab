# Phase 1: Basic Network Connectivity

## Goal

Verify basic network connectivity between the homelab devices.

## Network

- Network: 10.10.0.0/24
- Gateway: 10.10.0.1
- Router/firewall: VyOS/Zotac
- Managed switch: TP-Link TL-SG108E

## Current DNS Mode

At this stage, most systems use public DNS directly.

Current DNS servers in use:

- 1.1.1.1
- 8.8.8.8 where configured

Future target:

- Internal DNS resolver for homelab hostnames
- Public DNS used only as upstream resolvers

## Static Infrastructure Addresses

### Proxmox Hosts

- pve01
  - IP address: 10.10.0.10
  - Gateway: 10.10.0.1
  - DNS: 1.1.1.1
  - Address type: static

- pve02
  - IP address: 10.10.0.11
  - Gateway: 10.10.0.1
  - DNS: 1.1.1.1
  - Address type: static

- pve03
  - IP address:10.10.0.12
  - Gateway: 10.10.0.1
  - DNS: 1.1.1.1
  - Address type: static

### Management Interfaces

- ilosgh017t7n8
  - Role: HPE iLO management interface
  - IP address: 10.10.0.146
  - Gateway: 10.10.0.1
  - DNS: none currently configured

### Network Devices

- tl-sg108e
  - Role: TP-Link TL-SG108E managed switch
  - IP address: 10.10.0.100
  - Gateway: not documented
  - DNS: not documented

### Client Systems

- ws01
  - Role: HP Z2 Tower G4 workstation
  - IP address: 10.10.0.158
  - Gateway: 10.10.0.1
  - DNS: 1.1.1.1, 8.8.8.8

- lg-gram
  - Role: LG Gram laptop
  - IP address: 10.10.0.135
  - Gateway: 10.10.0.1
  - DNS: 1.1.1.1, 8.8.8.8

## Connectivity Validation

| Hostname | IP Address | Gateway Reachable | Internet IP Reachable | DNS Resolution |
|---|---:|---|---|---|
| pve01 | 10.10.0.10 | yes | yes | yes |
| pve02 | 10.10.0.11 | yes | yes | yes |
| pve03 | 10.10.0.12 | yes | yes | yes |
| ilosgh017t7n8 | 10.10.0.146 | yes | yes | no |
| tl-sg108e | 10.10.0.100 | yes | not tested / not required | not tested / not required |
| ws01 | 10.10.0.158 | yes | yes | yes |
| lg-gram | 10.10.0.135 | yes | yes | yes |

## Notes

- `pve01`, `pve02`, `ws01`, and `lg-gram` have full basic network connectivity.
- `ilosgh017t7n8` has IP connectivity but no DNS resolution because DNS is not configured.
- `tl-sg108e` is reachable for local management. Internet and DNS testing are not required at this stage.
- Internal DNS has not been implemented yet.
- `pve03` is a nested Proxmox VE node running as an Incus VM on `ws01`. It is
    reachable on the homelab LAN AT `10.10.0.12` and can reach the gateway, `pve   01` and `pve02` through a
     bridge on `ws01`. It is used for lab/testing only and
    is noet an independent physical failure domain.
