# Cable Map

## Purpose

This document records the verified physical network connections in the homelab.

It helps prevent guessing during network troubleshooting by showing which devices are connected, which switch ports are used, and which details still need verification.

## Network context

The homelab network is separate from the home network.

The VyOS router/firewall connects directly to the Internet on its WAN side and provides LAN connectivity to the homelab through the TP-Link managed switch.

## Verified connections

- Internet -> VyOS router/firewall WAN port
- VyOS router/firewall LAN port -> TP-Link TL-SG108E port 1
- TP-Link TL-SG108E port 2 -> Dell PowerEdge R610 iDRAC
- TP-Link TL-SG108E port 3 -> Dell PowerEdge R610 Ethernet port 2
- TP-Link TL-SG108E port 4 -> Samsung DeskTop System
- LG Gram  -> Not connected by Ethernet or Wi-Fi

## Device connection notes

### VyOS router/firewall

- WAN: connected directly to the Internet
- LAN: connected to TP-Link TL-SG108E port 1
- Role: router/firewall for the homelab network

### TP-Link TL-SG108E managed switch

- Port 1: connected to VyOS router/firewall LAN port
- Port 2: connected to Dell PowerEdge R610 iDRAC
- Port 3: connected to Dell PowerEdge R610 Ethernet port 2
- Port 4: connected to Samsung DeskTop System

### Dell PowerEdge R610

- iDRAC: connected to TP-Link TL-SG108E port 2
- Ethernet port 2: connected to TP-Link TL-SG108E port 3

### Samsung DeskTop System

- Connected to TP-Link TL-SG108E port 4

### LG Gram

- Not connected by Ethernet
- Not connected by Wi-Fi
- No active network connection documented in this phase

## Cable labels

All Ethernet cables are currently unlabeled.

This is a documentation and troubleshooting risk because cables may be harder to identify during changes, outages, or hardware moves.

## Unknowns / needs verification

- IP addresses are currently unknown.
- DHCP leases need to be checked on the VyOS router.
- TP-Link switch management IP is unknown.
- VyOS interface names need verification.
- Dell R610 Ethernet port 2 role needs verification.
- Cable labels need to be added or documented if labeling is not physically possible.

## Next verification steps

- Check DHCP leases on the VyOS router.
- Confirm the VyOS WAN and LAN interface names.
- Confirm the TP-Link switch management IP.
- Confirm whether Dell R610 Ethernet port 2 is the intended Proxmox management interface.
- Label important cables or create a consistent cable naming system.

## Change log

- Initial cable map created during Phase 0: Physical lab foundation.

