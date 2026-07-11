# Physical Layout

## Purpose

This file documents the physical layout of the homelab.

It records where each physical device is located, what role it has, whether it is powered on, and what still needs verification. This helps avoid confusion as the lab grows and makes future troubleshooting easier.

## Physical environment

The homelab equipment is located in the office.

The VyOS router/firewall provides the lab gateway and firewall role.

The TP-Link TL-SG108E switch is the central wired switch for the lab.

The HPE DL360 Gen10 is the main Proxmox virtualization host.

The Samsung DeskTop System has been repurposed as an additional Proxmox host.

The HP Z2 Tower G4 workstation, is the main control/admin node.

The ipTIME AX2004T is used as a Wi-Fi access point for the lab.

The LG Gram laptop is a secondary control/admin node.

The Dell PowerEdge R610 has been decommissioned and is no longer part of the active infrastructure.


## Devices

### VyOS router/firewall

- Device type: ZBOX mini PC
- OS/software: VyOS
- Powered on: Yes
- Physical location: On a table next to the air conditioner
- Network role: Router/firewall for the homelab network
- Notes: Provides the main lab gateway and firewall role

### TP-Link TL-SG108E

- Device type: Easy Smart Switch
- OS/software: TP-Link switch firmware
- Powered on: Yes
- Physical location: Mounted on the wall
- Network role: Central managed switch for homelab wired devices
- Notes: Management IP still needs verification

### HPE DL360 Gen10

- Device type: Rack server
- OS/software: Proxmox VE 9.2.4
- Powered on: Yes
- Physical location: Under a table in a separate area
- Network role: Main virtualization server
- Notes: 
    - Replaced the Dell PowerEdge R610 as the main Proxmox host
    - iLO is connected to the switch for out-of-band management
    - Proxmox/network interface is connected through Adapter 1, HPE Ethernet 1Gb 4-port 331i  Adapter, NIC port 1

### Samsung DeskTop System

- Device type: Desktop
- OS/software: Proxmox VE 9.2.4
- Powered on: Yes
- Physical location: Under the desk
- Network role: Additional Proxmox virtualization host
- Notes:
    - Previously used as the main admin/control desktop
    - Repurposed as Proxmox host

### HP Z2 Tower G4 Workstation

- Device type: Workstation
- OS/software: Ubuntu 26.04 LTS
- Powered on: Yes
- Physical location: Under the desk
- Network role: Main control/admin node
- Notes:
    - Connected by Ethernet
    - Used for SSH, documentation, Git, Proxmox/VyOS administration, and future automation work.
### ipTIME AX2004T

- Device type: Wireless router used as access point
- Powered on: Yes
- Network role: Wi-Fi access point for the homelab
- Notes:
    - Connected to the TP-Link switch through one of the ipTIME LAN ports
    - Should act as an access point/bridge, not as main router

### LG Gram

- Device type: Laptop
- OS/software: Rocky Linux 10.2
- Powered on: Yes
- Physical location: On the desk
- Network role: Secondary Linux client / Secondary control node
- Notes: 
    - Connected to Wi-Fi through ipTIME AX2004T

## Retired devices

### Dell PowerEdge R610

- Device type: Rack server
- Status: Decommissioned
- Previous role: Main Proxmox virtualization server
- Replaced by: HPE DL360 Gen10
- Notes: 
    - No longer part of active infrastructure
    - Should not be referenced as the current Proxmox host


## Physical topology

The Internet connects directly to the VyOS router/firewall.

The VyOS router/firewall LAN port connects to the TP-Link TL-SG108E switch port 1.


The TP-Link switch connects to the HPE DL360 Gen10, HP Z2 Tower G4 workstation, Samsung DeskTop System, and ipTIME AX2004T access point.

The LG Gram laptop connects to the lab network by Wi-Fi through the ipTIME AX2004T.

## Unknowns / needs verification

- Device IP addresses need revalidation after the hardware refresh.
- TP-Link switch management IP is unknown.
- ipTIME AX2004T access point/bridge mode needs verification.
- ipTIME DHCP server status needs verification.
- TP-Link ports 7 and 8 are currently unused.

## Next improvements

- Check DHCP leases on the VyOS router to identify device IP addresses.
- Find or set the TP-Link switch management IP.
- Confirm that LG Gram receives an IP address form the expected lab network.
- confirm that the ipTIME AX2004T is acting as an AP/bridge and not as a second router.
- Label Ethernet cables if future troubleshooting or equipment movement requires it.

## Change log

- Initial physical layout documented during Phase 0: Physical lab foundation.
- Updated after hardware refresh:
    - Dell PowerEdge R610 decommissioned.
    - HPE DL360 Gen10 added as main Proxmox host.
    - HP Z2 Tower G4 workstation added as main control node.
    - ipTIME AX2004T added as Wi-Fi access point.
    - LG Gram updated to Rocky Linux 10.2 and Wi-Fi connection through ipTIME AX2004T.
