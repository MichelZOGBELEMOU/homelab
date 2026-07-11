# Cable Map

## Purpose

This document records the verified physical network connections in the homelab.

It helps prevent guessing during network troubleshooting by showing which devices are connected, which switch ports are used, and which details still need verification.

## Network context

The homelab network is separate from the home network.

The VyOS router/firewall connects directly to the Internet on its WAN side and provides LAN connectivity to the homelab through the TP-Link managed switch.

The TP-Link TL-SG108E switch is the central wired switch.

The ipTIME AX2004T is used as Wi-Fi access point for the LG Gram laptop. It should not be treated as the main router/firewall.

## Verified connections

- Internet -> VyOS router/firewall WAN port
- VyOS router/firewall LAN port -> TP-Link TL-SG108E port 1
- TP-Link TL-SG108E port 2 -> HPE DL360 Gen10 iLO Dedicated Network Port
- TP-Link TL-SG108E port 3 -> HPE DL360 Gen10 Adapter 1 / HPE Ethernet 1Gb 4-port 331i Adapter / NIC port 1
- TP-Link TL-SG108E port 4 -> HP Z2 Tower G4 workstation
- TP-Link TL-SG108E port 5 -> ipTIME AX2004T LAN port 1
- TP-Link TL-SG108E port 6 -> Samsung Desktop System / Proxmox host
- LG Gram -> Wi-Fi through ipTIME AX2004T

### TP-Link TL-SG108E

- Port 1: VyOS router/firewall LAN
- Port 2: HPE DL360 Gen10 iLO Dedicated Network Port
- Port 3: HPE DL360 Gen10 Adapter 1 / HPE Ethernet 1Gb 4-port 331i Adapter / NIC port 1
- Port 4: HP Z2 Tower G4 workstation
- Port 5: ipTIME AX2004T LAN port 1
- Port 6: Samsung Desktop System / Proxmox host
- Port 7: Unused
- Port 8: Unused


## Device connection notes

### VyOS router/firewall

- WAN: connected directly to the Internet
- LAN: connected to TP-Link TL-SG108E port 1
- Role: router/firewall for the homelab network

### TP-Link TL-SG108E managed switch

- Connects router, Proxmox hosts, control node, and wireless AP
- Role: central wired switch for lab devices

### HPE DL360 Gen10

- iLO Dedicated Network Port: connect to TP-Link TL-SG108E port 2
- Adapter 1 / HPE Ethernet 1Gb 4-port 331i Adapter / NIC port 1: connected to TP-Link TL-SG108E port 3
- Role: main Proxmox virtualization host

### HP Z2 Tower G4 Workstation

- Connected by Ethernet to TP-Link TL-SG108E port 4
- Role: main control/admin node

### ipTIME AX2004T

- Connected from TP-Link TL-SG108E port 5 to ipTIME LAN port 1
- Role: Wi-Fi access point for the lab
- Important: should operate as AP/bridge, not as the main router

### Samsung DeskTop System

- Connected to TP-Link TL-SG108E port 6
- Role: additional Proxmox virtualization host

### LG Gram

- Connection: Wi-Fi through ipTIME AX200AT
- Role: secondary Linux client / secondary control node
### Dell PowerEdge R610
- Status: Decommissioned
- Current cabling: no active connection documented
- Previous role: main Proxmox virtualization host.
-Replaced by: HPE DL360 Gen10
## Cable labels

All Ethernet cables are currently unlabeled.

This is a documentation and troubleshooting risk because cables may be harder to identify during changes, outages, or hardware moves.

## Unknowns / needs verification

- IP addresses are currently unknown.
- DHCP leases need to be checked on the VyOS router.
- TP-Link switch management IP is unknown.
- VyOS interface names need verification.
- Cable labels need to be added or documented if labeling is not physically possible.

## Next verification steps

- Check DHCP leases on the VyOS router.
- Confirm the VyOS WAN and LAN interface names.
- Confirm the TP-Link switch management IP.
- Confirm that the HPE DL360 Gen10 iLO and Proxmox interfaces are reachable.
- Confirm that the HP Z2 Tower G4 workstation can reach the gateway and Proxmox hosts.
- Confirm that LG Gram receives a lab-network IP through the ipTIME AP.
- Confirm that the ipTIME AX2004T is not running a unwanted second DHCP/NAT network.
- Label important cables or create a consistent cable naming system.

## Change log

- Initial cable map created during Phase 0: Physical lab foundation.
- Updated after hardware refresh:
    - Dell PowerEdge R610 removed from active cabling.
    - HPE DL360 Gen10 added on TP-Link ports 2 and 3.
    - HP Z2 Tower G4 workstation added on TP-Link port 4.
    - ipTIME AX2004T added on TP-Link port 5.
    - Samsung Desktop System repurposed as Proxmox host on TP-Link port 6.
    - LG Gram connected by Wi-Fi through ipTIME AX2004T.
