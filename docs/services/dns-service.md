# DNS Service

## Service Summary

| Item | Value |
|---|---|
| Service name | Internal DNS |
| Primary host | dns01.home.lab |
| Primary IP | 10.10.20.10 |
| Secondary host | dns02.home.lab, planned after primary validation |
| Secondary IP | 10.10.20.11 |
| Network zone | SRV / VLAN 20 |
| Software | BIND9 |
| Operating system | Debian 13 |
| Status | dns01 base OS installed, BIND9 not installed yet |
| Phase | Phase 6 — DNS service |

## Purpose

The DNS service provides internal hostname resolution for the homelab.

It allows administrators and future services to use stable names such as `pve01.home.lab`, `dns01.home.lab`, and `backup01.home.lab` instead of memorizing IP addresses.

## Service Scope

In scope:

- Internal `home.lab` zone
- Static records for infrastructure hosts
- Static records for infrastructure services
- DNS forwarding or recursion for external names
- Basic DNS validation from an admin system
- Future secondary DNS using `dns02`

Out of scope:

- Public DNS hosting
- DDNS
- DHCP-DDNS integration
- TLS certificates
- Reverse proxy configuration
- Public-facing services

## Service Owner

| Item | Value |
|---|---|
| Owner | Michel |
| Role | Homelab infrastructure administrator |

## Primary Server

| Item | Value |
|---|---|
| Hostname | dns01 |
| FQDN | dns01.home.lab |
| IP address | 10.10.20.10/24 |
| Gateway | 10.10.20.1 |
| VLAN | VLAN 20 / SRV |
| Proxmox host | pve02 |
| OS | Debian 13 |
| DNS software | BIND9 |
| Status | Base OS installed and validated |

## Planned Secondary Server

| Item | Value |
|---|---|
| Hostname | dns02 |
| FQDN | dns02.home.lab |
| IP address | 10.10.20.11/24 |
| Gateway | 10.10.20.1 |
| VLAN | VLAN 20 / SRV |
| Proxmox host | pve03 / HP Z620 Workstation |
| OS | Debian 13 |
| DNS software | BIND9 |
| Status | Planned after dns01 validation |

## dns01 Base Install Validation

| Check | Result |
|---|---|
| Hostname is dns01 | Passed |
| Static IP is 10.10.20.10/24 | Passed |
| Gateway is 10.10.20.1 | Passed |
| VLAN is 20 / SRV | Passed |
| Can ping gateway 10.10.20.1 | Passed |
| Can ping 1.1.1.1 | Passed |
| Can resolve debian.org | Passed |
| Can SSH from admin machine | Passed |
| Proxmox host is pve02 | Passed |

Current status:

- Base OS installed and ready for BIND9 installation.

## dns01 Partition Layout

| Mount or Volume | Size | Type | Purpose |
|---|---:|---|---|
| /boot | 1 GB | ext4 | Boot files |
| / | 10 GB | LVM | Root filesystem |
| swap | 2 GB | LVM | Swap space |
| /var | 7 GB | LVM | Logs, package cache, and service data |

Operational note:

- A separate `/var` partition is useful for service logs and package/cache data.
- `/var` should be monitored after BIND9 is installed because DNS logs and package cache can grow over time.

## Service Dependencies

Required dependencies:

- Working Server VLAN gateway: `10.10.20.1`
- Working Admin-to-Server routing/firewall path
- Static IP address for `dns01`: `10.10.20.10`
- Upstream DNS path for external resolution
- Working SRV-to-WAN firewall policy for updates, DNS, and NTP

Related documentation:

- `docs/network/ip-plan.md`
- `docs/network/network-zones.md`
- `docs/network/current-firewall-policiy.md`
- `docs/network/dns-plan.md`
- `docs/network/dns-records.md`
- `docs/proxmox/vm-inventory.md`

## Users and Clients

Initial clients:

- Admin workstation on ADMIN VLAN
- Infrastructure hosts that need internal name resolution

Future clients:

- Server VLAN systems
- Client VLAN systems
- Guest/IoT systems only if specifically allowed

## Access Policy

Administrative access:

- ADMIN VLAN may administer `dns01` over SSH.

DNS query access:

- ADMIN VLAN may query `dns01`.
- SRV VLAN may query `dns01`.
- MGMT VLAN may query `dns01` if needed by infrastructure devices.
- CLIENT VLAN access is deferred until the Client VLAN is active.
- Guest/IoT DNS access is denied by default unless a future documented rule allows it.

WAN access:

- WAN must not initiate new connections to `dns01`.
- This is an internal DNS service only.

## Expected Ports

| Port | Protocol | Purpose |
|---:|---|---|
| 53 | UDP | Standard DNS queries |
| 53 | TCP | Large DNS responses, fallback DNS, and DNS operations when needed |
| 22 | TCP | SSH administration from ADMIN VLAN only |

Outbound from `dns01`:

| Port | Protocol | Purpose |
|---:|---|---|
| 53 | UDP/TCP | External DNS forwarding or recursion |
| 80 | TCP | Package repositories |
| 443 | TCP | Secure package repositories and downloads |
| 123 | UDP | NTP time synchronization |
| ICMP | ICMP | Troubleshooting |

## Availability Target

This is a homelab service, but it should be treated as important infrastructure.

Expected behavior:

- `dns01` should be reachable from approved VLANs.
- Internal records should resolve consistently.
- External resolution should continue to work through forwarding or recursion.
- Failure should be documented and troubleshot because DNS affects many future services.
- `dns02` will be added later on pve03 to practice secondary DNS, zone transfers, and resolver failover behavior.

## Backup and Recovery Notes

Backup requirements:

- BIND9 configuration files
- Zone files
- Service notes and validation results

Recovery expectation:

- A rebuilt `dns01` should be able to restore internal name resolution from documented configuration and records.

Backup implementation status:

- Deferred until backup phase

## Monitoring Notes

Future monitoring should check:

- DNS service process is running
- UDP/53 responds on `10.10.20.10`
- TCP/53 responds on `10.10.20.10`
- `dns01.home.lab` resolves correctly
- `pve01.home.lab` resolves correctly
- External lookup through DNS service works
- `/var` has enough free space

Example future checks:

- `dig @10.10.20.10 dns01.home.lab`
- `dig @10.10.20.10 pve01.home.lab`
- `dig @10.10.20.10 example.com`
- `df -h /var`

Monitoring implementation status:

- Deferred until monitoring phase

## Implementation Checklist

- [x] Create the `dns01` VM.
- [x] Configure static IP `10.10.20.10`.
- [x] Validate gateway reachability.
- [x] Validate Internet IP reachability.
- [x] Validate external DNS resolution.
- [x] Validate SSH from admin machine.
- [x] Install BIND9 on Debian 13.
- [ ] Configure the `home.lab` zone.
- [ ] Add core infrastructure records.
- [ ] Add service alias records.
- [ ] Configure upstream DNS forwarding or recursion.
- [ ] Validate internal DNS resolution.
- [ ] Validate external DNS resolution through BIND9.
- [ ] Record failures and unknowns.
- [ ] Build `dns02` on pve03 after `dns01` is validated.
- [ ] Configure and validate zone transfer from `dns01` to `dns02`.

## Validation Checklist

From an admin machine after BIND9 configuration:

- `dig @10.10.20.10 dns01.home.lab`
- `dig @10.10.20.10 edge.home.lab`
- `dig @10.10.20.10 pve01.home.lab`
- `dig @10.10.20.10 example.com`

Expected results:

- `dns01.home.lab` resolves to `10.10.20.10`.
- `edge.home.lab` resolves to the router/firewall management address.
- `pve01.home.lab` resolves to the Proxmox management address.
- External resolution still works.

## BIND9 Install Validation

| Check | Result |
|---|---|
| BIND9 package installed | Passed |
| bind9.service active | Passed |
| BIND version | BIND 9.20.26-1~deb13u1-Debian |
| BIND configuration syntax | Passed |

Validation commands:

```bash
systemctl is-active bind9.service
sudo named -v
sudo named-checkconf
```

Validation output:

```text
active
BIND 9.20.26-1~deb13u1-Debian (Stable Release) <id:>
```

Operational note:

- `named-checkconf` produced no output, which means the BIND9 configuration syntax passed.

