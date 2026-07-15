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
   |
   |-- Port 6
   |    |
   |    +-- Samsung Desktop System / Proxmox
   |        | Current IP: 10.10.0.11
   |        | Web UI: https://10.10.0.11:8006
   |
   |-- Port 5
   |    |
   |    +-- ipTIME AX2004T LAN port 1
   |         |
   |         +-- lg-gram: Wi-Fi
   |              | Interface: wlp1s0
   |              | Current IP: 10.10.0.135
   |-- Port 4
   |     |
   |     +-- HP Z2 Tower G4 workstation
   |         | Interface: eno1
   |         | Current IP: 10.10.0.158
   |         |
   |         +-- Incus VM: pv03
   |              | Role: nested Proxmox VE lab node
   |              | Curent IP: 10.10.0.12
   |
   |
   |-- Port 2
   |     | 
   |     +-- HPE DL360 Gen10 iLO Dedicated Network Port
   |         | Current IP: 10.10.0.146
   |
   |-- Port 3
         |
         +-- HPE DL360 Gen10 Adapter 1 / HPE Ethernet 1Gb 4-port 331i Adapter / NIC port 1
             | Current IP: 10.10.0.10
             | Web UI: https://10.10.0.10:8006
```


## Core Network

- Gateway/router: VyOS/Zotac
  - Gateway IP: 10.10.0.1

- Managed switch: TP-Link TL-SG108E
  - Hostname: tl-sg108e
  - Management IP: 10.10.0.100

## Connected Infrastructure Devices

- HPE DL360 Gen10 Proxmox host
  - Hostname: pve01
  - IP address: 10.10.0.10

- Samsung desktop Proxmox host
  - Hostname: pve02
  - IP address: 10.10.0.11

- HPE iLO management interface
  - Hostname: ilosgh017t7n8
  - IP address: 10.10.0.146

- Nested Proxmox lab node
  - Hostname: pve03
  - IP address: 10.10.0.12
  - Parent host: ws01 / HP Z2 Tower G4
  - Note: Proxmox cluster, quarum, and migration testing.
            Not an independent physical failure domain.

## Client Devices

- HP Z2 Tower G4 workstation
  - Hostname: ws01
  - IP address: 10.10.0.158

- LG Gram laptop
  - Hostname: lg-gram
  - IP address: 10.10.0.135
  - Connected through Wi-Fi via the ipTIME AX2004T access point

## Access Point

- ipTIME AX2004T access point
  - Connected to TP-Link TL-SG108E port 5
  - Used for wireless client access

