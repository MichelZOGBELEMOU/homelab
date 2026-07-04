# Current Network Topology

## Phase

Phase 1: Basic Network Connectivity

## Purpose

This document records the current verified homelab network topology.

It focuses on how the main devices are connected and how traffic flows through the lab network.

This is not the final network design. It represents the current working topology discovered during Phase 1.

## Topology diagram

```text
Internet
   |
   | WAN connection
   |
VyOS router/firewall
   | WAN: eth1
   | LAN: eth2 - 10.10.0.1/24
   |
   | LAN
   |
TP-Link TL-SG108E switch
   |
   |-- Port 4
   |     |
   |     +-- ubuntu-desktop
   |         | Interface: enp3s0
   |         | Current IP: 10.10.0.129
   |
   |-- Port 2
   |     | 
   |     +-- Dell PowerEdge R610 iDRAC
   |         | Current IP: 10.10.0.102
   |
   |-- Port 3
         |
         +-- Dell PowerEdge R610 / Proxmox
             | Current IP: 10.10.0.131
             | Web UI: https://10.10.0.131:8006
```


## Device roles

### VyOS router/firewall

- Role: lab router, firewall, DHCP provider, and gateway
- WAN interface: `eth1`
- LAN interface: `eth2`
- LAN IP address: `10.10.0.1/24`
- Default gateway for lab clients: `10.10.0.1`

### TP-Link TL-SG108E switch

- Role: central LAN switch
- Connects the router, admin desktop, and Dell R610 network interfaces
- Admin desktop is connected on switch port 4
- Switch management IP: unknown

### ubuntu-desktop

- Role: admin/control machine
- Connection type: Ethernet
- Network interface: `enp3s0`
- Current IP address: `10.10.0.129`
- Connected to switch port 4

### Dell PowerEdge R610 iDRAC

- Role: out-of-band server management
- Current IP address: `10.10.0.102`
- Connected to the LAN through the TP-Link switch Port 2
- Reachable from `ubuntu-desktop`

### Dell PowerEdge R610 / Proxmox host

- Role: virtualization host
- Current IP address: `10.10.0.131`
- Connected to the LAN through the TP-Link switch Port 3
- Proxmox web interface: `https://10.10.0.131:8006`
- Reachable from `ubuntu-desktop`

## Traffic path

### Admin desktop to router

```text
ubuntu-desktop
  -> TP-Link switch
  -> VyOS LAN interface eth2
  -> 10.10.0.1
```

### Admin desktop to Proxmox

```text
ubuntu-desktop
  -> TP-Link switch
  -> Dell PowerEdge R610 / Proxmox host
  -> 10.10.0.131
```

### Admin desktop to iDRAC

```text
ubuntu-desktop
  -> TP-Link switch
  -> Dell PowerEdge R610 iDRAC
  -> 10.10.0.102
```

## Current addressing status

The following addresses were observed during Phase 1:

- VyOS router LAN: `10.10.0.1/24`
- ubuntu-desktop: `10.10.0.129`
- R610 iDRAC: `10.10.0.102`
- Proxmox host: `10.10.0.131`

The admin desktop, iDRAC, and Proxmox addresses are currently assigned by DHCP and should be treated as current observed addresses, not permanent infrastructure assignments.

## Known unknowns

- TP-Link switch management IP address is not documented yet.

## Follow-up topology improvements

These should be handled in later phases:

- Create a stable IP address plan.
- Add DHCP reservations or static IPs for infrastructure devices.
- Document the switch management address.
- Add future network zones or VLANs after the basic network is stable.

## Summary

The current topology is a simple flat LAN:

```text
VyOS router/firewall -> TP-Link switch -> admin desktop / R610 iDRAC / Proxmox host
```

Basic connectivity is working, but infrastructure addressing still needs to be made stable in a later phase.

