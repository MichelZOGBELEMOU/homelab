# DHCP Plan

## Purpose

Plan the DHCP service for the segmented `home.lab` network.

This phase moves DHCP from basic router-provided addressing to dedicated Kea DHCP servers with scopes, reservations, relay, and high availability.

## Scope

In scope:

- Kea DHCPv4 service.
- DHCP primary and standby servers.
- DHCP scopes for selected VLANs.
- DHCP reservations for known clients.
- DNS and domain options for clients.
- VyOS DHCP relay.
- Basic failover validation.

Out of scope:

- DHCP-DDNS.
- External DDNS.
- Reverse proxy/TLS.
- Public DNS.
- DMZ DHCP.
- Full automation.

## DHCP servers

| Host | IP address | Role | VLAN | Proxmox host |
|---|---:|---|---|---|
| `dhcp01.servers.home.lab` | `10.10.20.14` | Primary | VLAN 20 / Servers | `pve03` |
| `dhcp02.servers.home.lab` | `10.10.20.15` | Standby | VLAN 20 / Servers | `pve01` |

DHCP servers are placed on different Proxmox hosts to reduce single-host failure risk.

## DHCP implementation

- Software: Kea DHCPv4
- HA mode: hot-standby
- DHCP service: `isc-kea-dhcp4-server.service`
- Configuration file: `/etc/kea/kea-dhcp4.conf`
- Lease database: `/var/lib/kea/kea-leases4.csv`

## DHCP relay

DHCP servers are in VLAN 20, so client VLANs need DHCP relay on VyOS.

Current validated relay:
service {
    dhcp-relay {
        listen-interface eth2.30
        server 10.10.20.14
        server 10.10.20.15
        upstream-interface eth2.20
    }

Meaning:

- Listen VLAN: VLAN 30 / Admin
- Upstream VLAN: VLAN 20 / Servers
- Relay targets:
  - `10.10.20.14`
  - `10.10.20.15`

## DHCP scopes

| VLAN | Zone | Subnet | Gateway | DHCP pool | Status |
|---:|---|---|---|---|---|
| 10 | Management | `10.10.10.0/24` | `10.10.10.1` | `10.10.10.100 - 10.10.10.150` | Pending validation |
| 20 | Servers | `10.10.20.0/24` | `10.10.20.1` | None | Static only |
| 30 | Admin | `10.10.30.0/24` | `10.10.30.1` | `10.10.30.100 - 10.10.30.200` | Validated |
| 40 | Clients | `10.10.40.0/24` | `10.10.40.1` | `10.10.40.2 - 10.10.40.100` | Pending validation |
| 50 | Guest/IoT | `10.10.50.0/24` | `10.10.50.1` | `10.10.50.2 - 10.10.50.100` | Pending validation |
| 60 | DMZ | `10.10.60.0/24` | `10.10.60.1` | None | Deferred |

## DHCP options

Clients should receive:

- DNS servers:
  - `10.10.20.12`
  - `10.10.20.13`
- Domain:
  - `home.lab`
- Router:
  - VLAN-specific gateway

## Reservations

Initial reservations:

| Device | VLAN | MAC address | Reserved IP | Status |
|---|---:|---|---|---|
| `ws01` | 30 | `fe:cd:7a:10:bf:52` | `10.10.30.10` | Validated |
| Admin host | 30 | `BC:24:11:30:EB:75` | `10.10.30.11` | Pending verification |

Detailed reservations are documented in:

- `docs/network/dhcp-reservations.md`

## High availability plan

Kea HA mode:

- Hot-standby

Expected behavior:

- `dhcp01` normally serves as primary.
- `dhcp02` remains standby.
- If `dhcp01` is unavailable, `dhcp02` should continue DHCP service.
- HA heartbeat should resume after `dhcp01` is restored.

HA communication:

- `dhcp01`: `http://10.10.20.14:8000/`
- `dhcp02`: `http://10.10.20.15:8000/`
- Required port: TCP `8000`

## Validation plan

Required checks:

- Kea configuration syntax passes.
- DHCP service is active.
- VyOS relay points to both DHCP servers.
- VLAN 30 client receives expected reservation.
- `dhcp02` allocates a lease during controlled `dhcp01` outage.
- HA heartbeat resumes after `dhcp01` is restored.

Completed validation:

- VLAN 30 reservation: PASS
- VLAN 30 relay: PASS
- DHCP HA failover on VLAN 30: PASS

Pending validation:

- VLAN 10 lease test.
- VLAN 40 lease test.
- VLAN 50 lease test.
- DNS option test.
- Domain option test.
- Full firewall rule evidence.

## Acceptance criteria

This phase is complete when:

- [x] DHCP primary server is planned.
- [x] DHCP standby server is planned.
- [x] DHCP scopes are defined.
- [x] DHCP reservations are defined.
- [x] VLAN 30 DHCP is validated.
- [x] DHCP HA failover is validated for VLAN 30.
- [ ] VLAN 10 DHCP is validated.
- [ ] VLAN 40 DHCP is validated.
- [ ] VLAN 50 DHCP is validated.
- [ ] DHCP relay/firewall policy is documented.

