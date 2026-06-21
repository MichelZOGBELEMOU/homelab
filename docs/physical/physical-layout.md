# Physical Layout

## Purpose

This file documents the physical layout of the homelab.

It records where each physical device is located, what role it has, whether it is powered on, and what still needs verification. This helps avoid confusion as the lab grows and makes future troubleshooting easier.

## Physical environment

The homelab equipment is located in the office.

The VyOS router/firewall is on a table next to the air conditioner. This location has been reviewed and is considered safe.

The TP-Link TL-SG108E switch is mounted on the wall.

The Dell PowerEdge R610 is under a table in a separate area.

The Samsung DeskTop System and LG Gram laptop are on the desk.

Cable organization is currently acceptable.

## Devices

### VyOS router/firewall

- Device type: ZBOX mini PC
- OS/software: VyOS
- Powered on: Yes
- Physical location: On a table next to the air conditioner
- Network role: Router/firewall for the homelab network
- Notes: Location reviewed and considered safe

### TP-Link TL-SG108E

- Device type: Easy Smart Switch
- OS/software: TP-Link switch firmware
- Powered on: Yes
- Physical location: Mounted on the wall
- Network role: Managed switch for homelab devices
- Notes: Management IP still needs verification

### Dell PowerEdge R610

- Device type: Rack server
- OS/software: Proxmox VE 9.2.3
- Powered on: Yes
- Physical location: Under a table in a separate area
- Network role: Main virtualization server
- Notes: Hosts the virtualization platform for lab services

### Samsung DeskTop System

- Device type: Desktop
- OS/software: Ubuntu 26.04 LTS
- Powered on: Yes
- Physical location: On the desk
- Network role: Main management node
- Notes: Connected to the TP-Link switch

### LG Gram

- Device type: Laptop
- OS/software: Ubuntu 26.04 LTS
- Powered on: Yes
- Physical location: On the desk
- Network role: Secondary management node
- Notes: Powered on but not currently connected to the network

## Physical topology

The Internet connects directly to the VyOS router/firewall.

The VyOS router/firewall LAN port connects to the TP-Link TL-SG108E switch.

The TP-Link switch connects to the Dell PowerEdge R610 and the Samsung DeskTop System.

The LG Gram laptop is physically present and powered on, but it is not currently connected to the network.

## Unknowns / needs verification

- Device IP addresses are unknown.
- TP-Link switch management IP is unknown.
- Exact power source for each device needs verification.
- LG Gram network connection method needs a decision: Wi-Fi or USB Ethernet adapter.

## Next improvements

- Record the power source for each device.
- Check DHCP leases on the VyOS router to identify device IP addresses.
- Find or set the TP-Link switch management IP.
- Decide whether the LG Gram should connect by Wi-Fi or USB Ethernet adapter.
- Label Ethernet cables if future troubleshooting or equipment movement requires it.

## Change log

- Initial physical layout documented during Phase 0: Physical lab foundation.

