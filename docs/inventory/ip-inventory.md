# IP Inventory

## Purpose

This document records the currently observed IP addresses in the homelab.

The goal is to make troubleshooting easier by documenting which device uses each address, what the address is used for, and whether the address needs later review.

## Network summary

- Main LAN: 10.10.0.0/24
- Default gateway: 10.10.0.1
- Main router/firewall: ZBOX-CI327NANO-GS-01 / VyOS
- DHCP source: VyOS router/firewall
- Main switch: TP-Link TL-SG108E
- Wi-Fi AP: ipTIME AX2004T in AP mode
- VLAN status: not active yet

## Current IP addresses

## 10.10.0.1

- Device: ZBOX-CI327NANO-GS-01 / VyOS
- Role: router/firewall, default gateway, DHCP server
- Platform: VyOS 2026-02 Circinus
- Management service: SSH
- Address purpose: main infrastructure gateway address
- Notes:
  - Main gateway for the homelab LAN.
  - Provides DHCP service for the homelab network.

## 10.10.0.10

- Device: HPE ProLiant DL360 Gen10 / pve01
- Role: main Proxmox virtualization host
- Platform: Proxmox VE 9.2.4
- Management service: Proxmox Web UI / SSH
- Address purpose: Proxmox host management
- Notes:
  - Main physical virtualization host.
  - Connected to TP-Link switch port 3.

## 10.10.0.11

- Device: Samsung desktop / pve02
- Role: secondary Proxmox virtualization host
- Platform: Proxmox VE 9.2.4
- Management service: Proxmox Web UI / SSH
- Address purpose: Proxmox host management
- Notes:
  - Secondary physical Proxmox host.
  - Connected to TP-Link switch port 6.

## 10.10.0.12

- Device: nested Proxmox / pve03
- Role: lab/testing Proxmox node
- Platform: Proxmox VE 9.2.4
- Management service: Proxmox Web UI / SSH
- Address purpose: nested Proxmox lab/testing management
- Notes:
  - Nested VM hosted through HP Z2 / ws01.
  - Reaches the homelab LAN through a bridge on HP Z2 / ws01.
  - Should not be treated as an independent physical failure domain.

## 10.10.0.100

- Device: TP-Link TL-SG108E
- Role: managed switch
- Platform: TP-Link firmware
- Management service: Web UI
- Address purpose: switch management
- Notes:
  - Main switch for the homelab LAN.
  - VLANs are not active yet.

## 10.10.0.135

- Device: LG Gram / lg-gram
- Role: secondary control/admin node
- Platform: Rocky Linux 10.2
- Management service: local login / SSH status not documented
- Address purpose: admin/client address
- Gateway: 10.10.0.1
- DNS: 1.1.1.1
- Notes:
  - Connected through ipTIME AX2004T Wi-Fi AP.
  - Receives normal homelab LAN connectivity through VyOS.
  - ipTIME does not act as a second router.

## 10.10.0.146

- Device: HPE iLO 5
- Role: out-of-band management for HPE ProLiant DL360 Gen10
- Platform: HPE iLO 5
- Management service: Web UI
- Address purpose: server hardware management
- Notes:
  - Provides hardware-level server management.
  - Should be protected carefully because it can control server power and remote console access.

## 10.10.0.158

- Device: HP Z2 / ws01
- Role: main admin/control node
- Platform: Ubuntu 26.04
- Management service: local login / SSH status not documented
- Address purpose: admin workstation address
- Notes:
  - Main control workstation.
  - Hosts nested Proxmox / pve03.
  - Connected to TP-Link switch port 4.

## 192.168.0.1/24

- Device: ipTIME AX2004T
- Role: Wi-Fi access point management
- Platform: ipTIME firmware
- Management service: Web UI
- Address purpose: AP management address
- Notes:
  - Device is in AP mode.
  - DHCP is disabled on the ipTIME device.
  - It does not act as a second router for the homelab.
  - Management IP is outside the main 10.10.0.0/24 LAN and should be reviewed in a later IP planning phase.

## Addressing notes

- The main homelab LAN uses 10.10.0.0/24.
- The default gateway is 10.10.0.1.
- VyOS is the DHCP server.
- ipTIME AX2004T is only an access point and does not provide DHCP.
- VLANs are not active yet.
- A formal stable IP plan will be created in a later phase.

## Follow-up items
- Confirm which addresses are static, DHCP reservations, or manually configured.
- Create a formal IP plan.
- Decide management address conventions.
- Review whether the ipTIME management IP should move into the 10.10.0.0/24 management plan.
- Document DNS behavior for all admin/control nodes.



