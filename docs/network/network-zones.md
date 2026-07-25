# Network Zones

## Purpose

This document defines the logical network zones used in the homelab.

The goal is to move from one flat LAN to separated network zones using VLANs, subnets, routing, and firewall policy.

## Current State

The current network is flat:

- Network: `10.10.0.0/24`
- Gateway: `10.10.0.1`
- VLANs: not active yet

This means management, admin, client, server, and lab traffic are not separated yet.

## Planned Zones

## Management Zone

- VLAN: 10
- Subnet: `10.10.10.0/24`
- Purpose: infrastructure management
- Devices:
  - VyOS management interface
  - TP-Link switch management
  - Proxmox host management
  - HPE iLO

Access rule:

- Only Admin VLAN should manage this zone.
- Client, Guest/IoT, and DMZ should not access this zone.

## Server Zone

- VLAN: 20
- Subnet: `10.10.20.0/24`
- Purpose: internal infrastructure services

Expected services:

- DNS
- DHCP 
- NTP
- monitoring
- logging
- backup
- Zammad

Access rule:

- Admin VLAN can manage servers.
- Client VLAN can access only required services.
- Guest/IoT access should be denied by default.
- DMZ access should be restricted.

## Admin Zone

- VLAN: 30
- Subnet: `10.10.30.0/24`
- Purpose: trusted admin/control systems

Devices:

- `ws01`
- `lg-gram` through the ipTIME AX2004T AP

Access rule:

- Admin VLAN may access Management and Server VLANs.
- Admin VLAN may access Proxmox, VyOS, switch, and iLO management interfaces.
- Admin VLAN should be protected more than normal Client VLAN.
- Devices on Admin VLAN should use strong authentication and should not be guest/untrusted systems.


## Client Zone

- VLAN: 40
- Subnet: `10.10.40.0/24`
- Purpose: normal client devices and test systems

Access rule:

- Client VLAN may access the internet.
- Client VLAN may access selected internal services such as DNS.
- Client VLAN should not manage Proxmox, VyOS, switch, or iLO.

Current note:

- No dedicated client access port or Wi-Fi SSID is assigned yet.
- Because the ipTIME AP does not support VLANs, it cannot provide both Admin Wi-Fi and Client Wi-Fi at the same time.



## Guest/IoT Zone

- VLAN: 50
- Subnet: `10.10.50.0/24`
- Purpose: guest, untrusted, temporary, or IoT devices

Access rule:

- Internet access only by default.
- No access to Management VLAN.
- No access to Server VLAN unless explicitly allowed.

## DMZ Zone

- VLAN: 60
- Subnet: `10.10.60.0/24`
- Purpose: future public-facing services

Possible future systems:

- reverse proxy
- public test web service

Access rule:

- DMZ should not freely initiate access to internal networks.
- Any DMZ-to-internal traffic must be explicitly justified.

