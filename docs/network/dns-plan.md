# DNS Plan

## Purpose

This document defines the internal DNS plan for the homelab.

The goal of Phase 6 is to provide predictable internal name resolution for infrastructure hosts and future services before adding DHCP-DDNS, public DNS, reverse proxy, TLS, or external access.

## Scope

In scope for this phase:

- Internal DNS namespace
- Primary DNS server
- Planned secondary DNS server
- Important host records
- Important service records
- Forward lookup validation
- Basic client DNS configuration notes
- Firewall requirements for DNS queries

Out of scope for this phase:

- Public DNS
- External DDNS
- DHCP-DDNS automation
- Reverse proxy
- TLS certificates
- Public web access

## Internal DNS Domain

Selected internal domain:

- `home.lab`

Operational notes:

- `home.lab` is for internal homelab DNS only.
- It should not be used for public DNS records.
- If a future conflict appears with external name resolution, the domain can be reviewed and migrated.

## DNS Software Decision

Selected DNS software:

- BIND9

Operating system:

- Debian 13

Reason:

- BIND9 provides strong Linux sysadmin learning evidence.
- It teaches zone files, service configuration, syntax checking, logs, reload behavior, and validation with DNS tools.
- It supports a future primary/secondary DNS design using zone transfers.

## DNS Server Design

### Primary DNS Server

| Item | Value |
|---|---|
| Hostname | dns01 |
| FQDN | dns01.home.lab |
| Role | Primary internal DNS server |
| Proxmox host | pve02 |
| Operating system | Debian 13 |
| DNS software | BIND9 |
| VLAN | VLAN 20 / SRV |
| IP address | 10.10.20.10 |
| Gateway | 10.10.20.1 |
| Status | Base OS installed and validated |

### Secondary DNS Server

| Item | Value |
|---|---|
| Hostname | dns02 |
| FQDN | dns02.home.lab |
| Role | Secondary internal DNS server |
| Proxmox host | pve03 / HP Z620 Workstation |
| Operating system | Debian 13 |
| DNS software | BIND9 |
| VLAN | VLAN 20 / SRV |
| IP address | 10.10.20.11 |
| Gateway | 10.10.20.1 |
| Status | Planned after dns01 is built and validated |

## Network Placement

DNS belongs in the Server zone.

| Item | Value |
|---|---|
| Zone | SRV |
| VLAN | 20 |
| Subnet | 10.10.20.0/24 |
| Gateway | 10.10.20.1 |
| Primary DNS server IP | 10.10.20.10 |
| Secondary DNS server IP | 10.10.20.11 |

## Related IP Plan

Server VLAN planned addresses:

| IP Address | Hostname | Purpose | Status |
|---|---|---|---|
| 10.10.20.10 | dns01 | Primary internal DNS server | Base OS installed |
| 10.10.20.11 | dns02 | Secondary internal DNS server | Planned |
| 10.10.20.12 | ntp01 | Future NTP/time service | Deferred |
| 10.10.20.20 | monitor01 | Future monitoring service | Deferred |
| 10.10.20.21 | log01 | Future logging service | Deferred |
| 10.10.20.30 | backup01 | Future backup service | Deferred |
| 10.10.20.40 | zammad01 | Future ticketing/service desk | Deferred |
| 10.10.20.50 | dhcp01 | Future DHCP service if moved from VyOS | Deferred |

## Client Query Policy

Initial allowed DNS clients:

- ADMIN VLAN to `dns01`
- MGMT VLAN to `dns01` if infrastructure devices need internal DNS
- SRV VLAN to `dns01` for service-to-service resolution

Later allowed DNS clients:

- CLIENT VLAN after client network exists
- Guest/IoT only if there is a documented reason

Default rule:

- DNS queries should be allowed only to the approved internal DNS server.
- Inter-VLAN DNS access should be documented in the firewall policy before enabling broad access.

## Firewall Requirements

Required DNS traffic to `dns01`:

| Source Zone | Destination | Protocol/Port | Reason | Status |
|---|---|---|---|---|
| ADMIN | 10.10.20.10 | UDP/53 | DNS queries | Rule active, BIND9 validation pending |
| ADMIN | 10.10.20.10 | TCP/53 | Large DNS responses and DNS fallback | Rule active, BIND9 validation pending |
| MGMT | 10.10.20.10 | UDP/53 | Infrastructure DNS queries | Planned |
| SRV | 10.10.20.10 | UDP/53 | Server DNS queries | Planned |

Administrative access to `dns01`:

| Source Zone | Destination | Protocol/Port | Reason | Status |
|---|---|---|---|---|
| ADMIN | 10.10.20.10 | TCP/22 | SSH administration | Active / validated |

Required outbound traffic from `dns01`:

| Source | Destination Zone | Protocol/Port | Reason | Status |
|---|---|---|---|---|
| dns01 | WAN | UDP/53 | External DNS queries or forwarding | Active / validated |
| dns01 | WAN | TCP/53 | Large DNS responses and DNS fallback | Active / validated |
| dns01 | WAN | TCP/80 | Package repositories | Active / validated |
| dns01 | WAN | TCP/443 | Secure package repositories and downloads | Active / validated |
| dns01 | WAN | UDP/123 | NTP time synchronization | Active / validated |
| dns01 | WAN | ICMP | Troubleshooting | Active / validated |

WAN-to-SRV policy:

- No new inbound WAN-to-SRV traffic is allowed.
- This DNS service is internal only.
- `dns01` must not be exposed as public DNS.

SRV-to-ADMIN policy:

- No new SRV-to-ADMIN connections are allowed by default.
- Only established and related return traffic should be allowed.

## Forwarding Behavior

Internal DNS should resolve:

- `*.home.lab` internal records locally
- external domains through upstream DNS forwarding or recursion

Upstream DNS decision:

- Status: Needs decision
- Possible upstreams:
  - VyOS router/firewall
  - ISP DNS
  - public resolver

Recommended first choice:

- Use a known external resolver during initial validation.
- Later decide whether VyOS should be the upstream resolver.

## Reverse DNS

Reverse DNS is useful but not required for the first working DNS milestone.

Initial status:

- Reverse lookup: Deferred / needs decision

Future reverse zones may include:

- 10.10.10.0/24 — MGMT
- 10.10.20.0/24 — SRV
- 10.10.30.0/24 — ADMIN
- 10.10.40.0/24 — CLIENT

## Current Implementation Status

- `dns01` base Debian 13 installation completed.
- `dns01` is connected to VLAN 20 / SRV.
- `dns01` uses static IP `10.10.20.10/24`.
- Gateway `10.10.20.1` is reachable.
- Internet IP connectivity works.
- External DNS resolution works.
- SSH administration from ADMIN to `dns01` works.
- BIND9 installation is the next step.

## Implementation Milestones

- [x] Create `dns01` in the Server VLAN on pve02.
- [x] Configure static IP `10.10.20.10`.
- [x] Validate gateway reachability.
- [x] Validate Internet IP reachability.
- [x] Validate external DNS resolution.
- [x] Validate SSH from ADMIN to `dns01`.
- [x] Install BIND9 on `dns01`.
- [ ] Configure `home.lab` internal zone.
- [ ] Add core host records.
- [ ] Add core service records.
- [ ] Configure upstream forwarding or recursion.
- [ ] Validate internal resolution.
- [ ] Validate external resolution through BIND9.
- [ ] Commit DNS documentation.
- [ ] Build `dns02` on pve03 after `dns01` is validated.
- [ ] Configure and test BIND9 zone transfer from `dns01` to `dns02`.

## Validation Commands

Query DNS server directly after BIND9 is configured:

- `dig @10.10.20.10 dns01.home.lab`
- `dig @10.10.20.10 edge.home.lab`
- `dig @10.10.20.10 pve01.home.lab`
- `dig @10.10.20.10 example.com`

Check client resolver path:

- `resolvectl status`

## Unknowns

- Whether reverse DNS will be configured in Phase 6
- Whether `dns01` will forward through VyOS or directly to upstream resolvers
- Final VyOS firewall rule IDs/names in VyOS
- Exact VM ID and resource allocation for `dns01`
- Exact VM ID and resource allocation for future `dns02`

