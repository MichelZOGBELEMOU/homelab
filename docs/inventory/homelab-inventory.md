# Homelab Inventory

## Purpose

This document records the currently observed devices in the homelab.

The goal is to make troubleshooting easier by documenting which devices exist, what their roles are, how they are connected, and what hardware or platform facts are currently known.

## Devices

## ZBOX-CI327NANO-GS-01 / VyOS

- Device type: mini PC
- Physical or VM: physical
- Main role: router/firewall
- Operating system/platform: VyOS 2026-02 Circinus
- IP address: 10.10.0.1
- Management access: SSH
- Connected to:
  - eth2 LAN -> TP-Link switch port 1
  - eth1 WAN -> internet

### Hardware specifications

- CPU: Intel Celeron N3450 @ 1.10GHz, 4 cores
- RAM: 8 GB
- Storage: 120 GB SSD
- Network interfaces:
  - eth1: WAN
  - eth2: LAN
  - wlan0: down
- Other: none documented

### Known facts

- Router/firewall for the homelab network.
- Default gateway for the homelab LAN.
- DHCP server for the homelab network.

### Notes

- Firewall rules are not fully documented yet.

## TP-Link TL-SG108E

- Device type: managed switch
- Physical or VM: physical
- Main role: Layer 2 switch for the homelab LAN
- Operating system/platform: TP-Link firmware
- IP address: 10.10.0.100
- Management access: Web UI
- Connected to:
  - VyOS router/firewall
  - HPE ProLiant DL360 Gen10 / pve01
  - HPE iLO
  - Samsung desktop / pve02
  - HP Z2 / ws01
  - ipTIME AX2004T

### Hardware specifications

- Model: TP-Link TL-SG108E
- Ports: 8-port Gigabit Ethernet switch

### Known facts

- Provides switching for the current homelab LAN.
- VLANs are not active yet.

### Notes

- Current network is flat with no VLAN separation yet.

## HPE ProLiant DL360 Gen10 / pve01

- Device type: rack server
- Physical or VM: physical
- Main role: main physical virtualization host
- Operating system/platform: Proxmox VE 9.2.4
- IP address: 10.10.0.10
- Management access: Web UI / SSH
- Connected to: TP-Link switch port 3

### Hardware specifications

- CPU: 2 x Intel Xeon Silver 4110 @ 2.10GHz, 8 cores each
- RAM: 32 GB
- Storage:
  - 250 GB SSD
  - 500 GB HDD
  - 500 GB HDD
- Network interfaces:
  - 4 x 1 Gb Ethernet ports
  - 2 x 10 Gb Ethernet ports
  - 1 x dedicated Ethernet port for iLO
- Other: HPE iLO 5 out-of-band management

### Known facts

- Main virtualization host of the homelab.
- Replaced the retired Dell R610 as the main server platform.

### Notes

- Backup, recovery, and monitoring responsibilities are not fully documented yet.

## HPE iLO 5

- Device type: out-of-band management controller
- Physical or VM: physical management controller
- Main role: hardware-level management for HPE ProLiant DL360 Gen10
- Operating system/platform: HPE iLO 5
- IP address: 10.10.0.146
- Management access: Web UI
- Connected to: TP-Link switch port 2

### Hardware specifications

- Model: HPE iLO 5
- Interface: dedicated Ethernet management port

### Known facts

- Provides out-of-band management for the HPE ProLiant DL360 Gen10.
- Can be used for server power control and remote console access.

### Notes

- iLO access policy is not fully documented yet.
- This interface should be protected because it can control physical server access.

## Samsung desktop / pve02

- Device type: desktop computer
- Physical or VM: physical
- Main role: secondary Proxmox virtualization host
- Operating system/platform: Proxmox VE 9.2.4
- IP address: 10.10.0.11
- Management access: Web UI / SSH
- Connected to: TP-Link switch port 6

### Hardware specifications

- CPU: Intel Core i7-2600 @ 3.40GHz, 4 cores
- RAM: 12 GB
- Storage:
  - 120 GB SSD
  - 1 TB HDD
- Network interfaces:
  - 1 x 1 Gb Ethernet port
- Other: none documented

### Known facts

- Secondary virtualization host.

### Notes

- Final workload role is not fully documented yet.

## HP Z2 / ws01

- Device type: desktop workstation
- Physical or VM: physical
- Main role: admin/control node
- Operating system/platform: Ubuntu 26.04
- IP address: 10.10.0.158
- Management access: local login / SSH status not documented
- Connected to: TP-Link switch port 4

### Hardware specifications

- CPU: Intel Xeon E-2144G @ 3.60GHz, 4 cores
- RAM: 16 GB
- Storage:
  - 250 GB SSD
  - 500 GB HDD
- Network interfaces:
  - Ethernet interface connected to TP-Link switch
- Other:
  - Hosts nested Proxmox / pve03

### Known facts

- Main admin/control workstation.
- Used to manage and operate the homelab.
- Hosts the nested Proxmox lab/testing VM.

### Notes

- Admin security baseline is not fully documented yet.


## Nested Proxmox / pve03

- Device type: nested virtualization host
- Physical or VM: VM
- Main role: lab/testing Proxmox node
- Operating system/platform: Proxmox VE 9.2.4
- IP address: 10.10.0.12
- Management access: Web UI / SSH / Incus host access
- Connected to: homelab LAN through a bridge on HP Z2 / ws01

### Hardware specifications

- CPU: host CPU
- RAM: 4 GB
- Storage: 100 GB LVM
- Network interfaces:
  - NIC0
- Other: nested Proxmox VM

### Known facts

- Nested Proxmox VE node.
- Used for lab and testing.

### Notes

- Should be used for lab/testing only.
- Should not be treated as an independent physical failure domain.

## LG Gram / lg-gram

- Device type: laptop
- Physical or VM: physical
- Main role: secondary control/admin node
- Operating system/platform: Rocky Linux 10.2
- IP address: 10.10.0.135
- Management access: local login / SSH status not documented
- Connected to: ipTIME AX2004T by Wi-Fi

### Hardware specifications

- CPU: Intel Core i3-7100U @ 2.40GHz, 2 cores
- RAM: 12 GB
- Storage: 250 GB SSD
- Network interfaces:
  - wlp1s0
- Other: none documented

### Known facts

- Connects to the homelab network over Wi-Fi.
- Uses the VyOS gateway at 10.10.0.1.
- Uses DNS server 1.1.1.1.

### Notes

- Wi-Fi dependency may affect reliability for administration tasks.

## ipTIME AX2004T

- Device type: wireless router/access point
- Physical or VM: physical
- Main role: Wi-Fi access point
- Operating system/platform: ipTIME firmware
- IP address: 192.168.0.1/24 for management
- Management access: Web UI
- Connected to: TP-Link switch port 5

### Hardware specifications

- Model: ipTIME AX2004T
- Ports:
  - 4 x LAN ports
  - 1 x WAN port
- Wireless:
  - 2.4 GHz Wi-Fi
  - 5 GHz Wi-Fi

### Known facts

- Operates in AP mode.
- DHCP is disabled on the ipTIME device.
- Provides Wi-Fi access for the LG Gram.
- Does not act as a second router in the homelab.

### Notes

- Management IP is currently 192.168.0.1/24.
- Long-term AP management addressing should be reviewed during the IP planning phase.

