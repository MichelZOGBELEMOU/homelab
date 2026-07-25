# Network Change and Rollback Plan

## Purpose

This document defines the safety plan for VLAN migration.

Network changes can cause lockout. This plan keeps changes small, testable, and reversible.

## Current Working State

Current working network:

- Subnet: `10.10.0.0/24`
- Gateway: `10.10.0.1`
- Router: VyOS `edge`
- Switch: TP-Link TL-SG108E
- Switch management IP: `10.10.0.100`
- Admin workstation: `ws01` / `10.10.0.158`
- pve01: `10.10.0.10`
- pve02: `10.10.0.11`
- pve03: `10.10.0.12`
- HPE iLO: `10.10.0.146`

## Pre-change Checklist

Before making VLAN changes:

- [ ] Confirm physical access to router and switch.
- [ ] Confirm access to VyOS SSH from `ws01`.
- [ ] Confirm access to TP-Link web UI from `ws01`.
- [ ] Confirm access to pve01 web UI from `ws01`.
- [ ] Confirm current switch port connections.
- [ ] Save/export current TP-Link switch configuration if possible.
- [ ] Save VyOS configuration.
- [ ] Copy current Proxmox `/etc/network/interfaces`.
- [ ] Keep one emergency switch port available.
- [ ] Make only one small change at a time.

## Validation After Each Change

Run from `ws01` where possible:

- [ ] Ping gateway.
- [ ] Ping switch management IP.
- [ ] Ping pve01.
- [ ] Ping pve02.
- [ ] Ping pve03.
- [ ] Test internet access.
- [ ] Test DNS resolution.
- [ ] Test VyOS SSH.
- [ ] Test Proxmox web UI.
- [ ] Test TP-Link web UI.

## Rollback Principles

If access is lost:

1. Stop making additional changes.
2. Use physical access if needed.
3. Reconnect through emergency access port.
4. Revert the last switch VLAN change.
5. Revert the last VyOS VLAN/interface change.
6. Restore previous Proxmox network file if needed.
7. Confirm `10.10.0.0/24` access works again.

## Emergency Access Plan

Emergency port:

- Switch port: 8
- Purpose: recovery access
- Planned mode: access port
- VLAN: legacy network or management VLAN during migration

## Known Risks

- Moving switch management IP can lock out web UI.
- Moving Proxmox management IP can lock out Proxmox.
- Misconfigured trunk tagging can break router-switch communication.
- pve03 depends on ws01 networking and is not an independent physical host.

