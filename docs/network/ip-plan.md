# IP Plan

## Purpose

This document defines the planned IP addressing scheme for the homelab VLAN network.

The previous flat network was:

- Network: `10.10.0.0/24`
- Gateway: `10.10.0.1`
- DHCP range: `10.10.0.100-10.10.0.199`

The new design separates management, servers, admin systems, clients, guest/IoT devices, and future DMZ services.

## VLAN Subnets

### VLAN 10 — Management

- Subnet: `10.10.10.0/24`
- Gateway: `10.10.10.1`
- Purpose: infrastructure management
- Devices:
  - VyOS management
  - TP-Link switch management
  - Proxmox host management
  - HPE iLO

Planned addresses:

- `10.10.10.1` — `edge` / VyOS management gateway
- `10.10.10.16` — `tl-sg108e` / TP-Link switch
- `10.10.10.10` — `pve01`
- `10.10.10.11` — `pve02`
- `10.10.10.12` — `pve03`
- `10.10.10.146` — HPE iLO

### VLAN 20 — Servers

- Subnet: `10.10.20.0/24`
- Gateway: `10.10.20.1`
- Purpose: infrastructure service VMs

Planned addresses:

- `10.10.20.10` — `dns01`
- `10.10.20.11` — `dhcp01` if DHCP is later moved from VyOS
- `10.10.20.12` — `ntp01`
- `10.10.20.20` — `monitor01`
- `10.10.20.21` — `log01`
- `10.10.20.30` — `backup01`
- `10.10.20.40` — `zammad01`

### VLAN 30 — Admin

- Subnet: `10.10.30.0/24`
- Gateway: `10.10.30.1`
- Purpose: trusted administrator/control devices

Planned devices:

- `ws01`

### VLAN 40 — Clients

- Subnet: `10.10.40.0/24`
- Gateway: `10.10.40.1`
- Purpose: normal client devices and test endpoints

### VLAN 50 — Guest/IoT

- Subnet: `10.10.50.0/24`
- Gateway: `10.10.50.1`
- Purpose: guest, untrusted, temporary, or IoT devices

### VLAN 60 — DMZ

- Subnet: `10.10.60.0/24`
- Gateway: `10.10.60.1`
- Purpose: future reverse proxy and public-facing services

## Legacy Network

During migration, the existing flat network remains:

- Legacy subnet: `10.10.0.0/24`
- Legacy gateway: `10.10.0.1`
- Purpose: temporary rollback/migration network

## Addressing Rules

- Network infrastructure uses low static addresses.
- Proxmox hosts use predictable management addresses.
- Service VMs use static or reserved addresses.
- DHCP ranges should avoid infrastructure static addresses.
- Management, admin, and client networks must remain separate.

