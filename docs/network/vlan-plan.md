# VLAN Plan

## Purpose

This document defines the planned VLANs for the homelab network.

## VLAN Summary

| VLAN ID | Name | Subnet | Gateway | Purpose |
|---:|---|---|---|---|
| 10 | Management | `10.10.10.0/24` | `10.10.10.1` | Infrastructure management |
| 20 | Servers | `10.10.20.0/24` | `10.10.20.1` | Internal service VMs |
| 30 | Admin | `10.10.30.0/24` | `10.10.30.1` | Admin/control systems |
| 40 | Clients | `10.10.40.0/24` | `10.10.40.1` | Normal client devices |
| 50 | Guest/IoT | `10.10.50.0/24` | `10.10.50.1` | Untrusted/temporary devices |
| 60 | DMZ | `10.10.60.0/24` | `10.10.60.1` | Future public-facing services |

## Implementation Order

Do not configure all VLANs at once.

### Batch 1

Design only:

- IP plan
- hostname standard
- zone definitions
- switch port map
- rollback plan

### Batch 2

Create first VLANs:

- VLAN 10 — Management
- VLAN 20 — Servers

Keep the current flat `10.10.0.0/24` network available as a rollback/migration path.

### Batch 3

Configure switch VLAN behavior carefully:

- router-switch trunk
- Proxmox trunk if safe
- emergency access port

### Batch 4

Move selected infrastructure management addresses.

### Batch 5

Add Admin and Client VLANs.

### Batch 6

Add Guest/IoT and DMZ later.

## Current Wireless Limitation

The ipTIME AX2004T access point does not support VLAN tagging.

Current decision:

- Use ipTIME as Admin Wi-Fi only.
- Place the AP switch port in VLAN 30.
- Put `lg-gram` in the Admin VLAN.

A separate VLAN-capable AP or additional access point would be needed later for separate Client or Guest Wi-Fi.


## Safety Rules

- Do not move the admin workstation first.
- Do not move all Proxmox management interfaces at once.
- Keep one emergency access path.
- Record the current working configuration before changes.
- Test after every small change.
- Roll back immediately if management access is lost.

