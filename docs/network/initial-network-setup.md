# Phase 1: Basic Network Connectivity

## Goal

Verify basic network connectivity between the admin desktop, VyOS router, TP-Link switch, Dell PowerEdge R610 iDRAC, and Proxmox host.

## Current network facts

- Router / default gateway: `10.10.0.1`
- LAN subnet: `10.10.0.0/24`
- DHCP range: `10.10.0.100 - 10.10.0.199`
- DNS server provided by DHCP: `10.10.0.1`
- Admin desktop:
  - Hostname: `ubuntu-desktop`
  - Connection: Ethernet
  - Interface: `enp3s0`
  - Current IP: `10.10.0.129`
- Dell PowerEdge R610 iDRAC:
  - Current IP: `10.10.0.102`
  - Web UI reachable: yes
- Proxmox host:
  - Current IP: `10.10.0.131`
  - Web UI reachable: yes
  - URL tested: `https://10.10.0.131:8006`

## Verified connectivity tests

- Admin desktop can reach gateway `10.10.0.1`.
- Admin desktop can reach public IP `1.1.1.1`.
- Admin desktop can resolve and reach `google.com`.
- Admin desktop can reach R610 iDRAC.
- Admin desktop can reach Proxmox web UI.
- Admin desktop can reach iDRAC web UI.

## VyOS DHCP configuration summary

- DHCP server: enabled
- DHCP subnet: `10.10.0.0/24`
- DHCP range: `10.10.0.100 - 10.10.0.199`
- Default router option: `10.10.0.1`
- Domain option: `home.lab`
- DNS option: `10.10.0.1`
- Lease time: `86400` seconds

## Current risk

Important infrastructure devices currently use DHCP addresses inside the dynamic DHCP range:

- iDRAC: `10.10.0.102`
- Proxmox: `10.10.0.131`
- Admin desktop: `10.10.0.129`

These addresses may change after lease renewal or device replacement unless DHCP reservations or static addresses are configured.

## Future improvements

- Create DHCP reservations for infrastructure devices.
- Decide which devices should use static IP addresses.
- Document an IP address plan.
- Document switch management IP if available.
- Separate iDRAC management access from Proxmox host access.
- Later phases should handle IP planning, DHCP reservations, DNS, and naming.

## What I learned

Basic network connectivity should be verified layer by layer instead of guessed.

The correct order is:

1. Physical connection
2. Interface/IP address
3. Default gateway
4. Local network reachability
5. Internet reachability
6. DNS resolution
7. Service/web UI reachability

A working ping or web UI is useful evidence, but infrastructure addresses must be documented as dynamic, reserved, or static.

