# Router/Firewall Foundation

## Purpose

This document records the current router/firewall foundation for the homelab.

The VyOS router named `edge` currently provides the main gateway, DHCP, NAT, and SSH management functions for the lab network.

This document focuses on verified current state, known risks, and future improvements. It does not describe a completed firewall policy yet.

## Device summary

- Hostname: `edge`
- Platform: VyOS
- Version: VyOS 2026.02 circinus
- Role: Lab edge router/firewall
- WAN interface: `eth1`
- LAN interface: `eth2`

## Interface summary

### WAN

- Interface: `eth1`
- IP method: DHCP
- WAN IP status: IP address present
- Public WAN address: redacted
- Interface status: up

### LAN

- Interface: `eth2`
- LAN IP address: `10.10.0.1`
- LAN subnet: `10.10.0.0/24`
- Interface status: up

## Gateway and routing

The router provides the default gateway for the lab LAN.

- Default route interface: `eth1`
- Lab default gateway: `10.10.0.1`
- LAN clients can reach the internet: yes

The public WAN address is intentionally not documented in this repository.

## DHCP service

DHCP is enabled for the lab LAN.

- DHCP subnet: `10.10.0.0/24`
- DHCP range: `10.10.0.100-10.10.0.199`
- Gateway option: `10.10.0.1`
- DNS option: `1.1.1.1`
- Domain option: `home.lab`

Current DHCP leases were not captured during this validation step.

## NAT service

Source NAT is enabled for lab clients.

```text
text
Source network: 10.10.0.0/24
Outbound interface: eth1
Translation: masquerade
```

## SSH management

SSH management is enabled on the LAN interface only.

- Listen address: `10.10.0.1`
- Port: `22`
- Password authentication: disabled
- Default `vyos` user: disabled
- Administrative login: `sysadmin`

SSH configuration summary:

```text
text
ssh {
    disable-password-authentication
    listen-address 10.10.0.1
    port 22
}


## Firewall status

A formal firewall policy is not configured yet.

This means the router currently provides routing, DHCP, NAT, and management access, but traffic-filtering policy still needs to be designed and implemented.

## Known risks

- No formal firewall policy is configured yet.
- The router is a single point of failure for lab internet access.
- WAN address is provided by DHCP and may change.
- Lab clients currently use external DNS server `1.1.1.1` directly.
- Router configuration backup/restore has not been validated yet.
- Current DHCP leases were not captured in this validation step.

## Future improvements

- Define a basic WAN-to-LAN and LAN-to-router firewall policy.
- Document allowed management access.
- Capture current DHCP leases during validation.
- Create and test a router configuration backup.
- Decide whether DNS should remain external or later move to an internal DNS service.
- Add router monitoring checks in the monitoring phase.
- Add router log collection in the logging phase.

## Validation evidence

The following checks were performed:

- Verified WAN interface is up.
- Verified LAN interface is up.
- Verified WAN interface has an IP address.
- Verified default route uses `eth1`.
- Verified LAN clients can reach the internet.
- Verified DHCP range and options.
- Verified source NAT masquerade rule.
- Verified SSH listens on `10.10.0.1`.
- Verified SSH password authentication is disabled.
- Verified no formal firewall policy is configured yet.

