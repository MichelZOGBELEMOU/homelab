# DNS Plan

## Purpose

This document defines the internal DNS plan for the homelab.

Phase 6 provides predictable internal name resolution for infrastructure hosts and future services before adding DHCP-DDNS, public DNS, reverse proxy, TLS, or external access.

## Scope

In scope for this phase:

- Internal DNS namespace for the homelab.
- Authoritative DNS servers for internal zones.
- Client resolver pair for normal workstation/server DNS use.
- Forward zones for network/service areas.
- Reverse zones for VLAN subnets.
- Static records for core infrastructure hosts and DNS resolver hosts.
- Client-side validation from `ws01`.
- Basic troubleshooting documentation for DNS record mistakes.

Out of scope for this phase:

- Public DNS hosting.
- External DDNS.
- DHCP-DDNS automation.
- Reverse proxy.
- TLS certificates.
- Public web access.
- Application deployment records.

## Internal DNS Domain

Selected internal domain:

- `home.lab`

Operational notes:

- `home.lab` is for internal homelab DNS only.
- Internal names should not be published to public DNS.
- Public DNS, DDNS, reverse proxy, and TLS work will be handled in later phases.

## DNS Architecture

The DNS design separates authoritative DNS from client resolver service.

Authoritative DNS:

- `dns01.home.lab`
  - Role: Primary authoritative DNS server.
  - Software: BIND9.
  - Zone type: primary.
  - Zone files: `/var/lib/bind/`.

- `dns02.home.lab`
  - Role: Secondary authoritative DNS server.
  - Software: BIND9.
  - Zone type: secondary.
  - Zone files: `/var/cache/bind/secondaries/`.
  - Purpose: secondary zone loading, zone-transfer practice, and authoritative redundancy.

Client resolvers:

- `dns-resolv-01.servers.home.lab`
  - IP address: `10.10.20.12`.
  - Role: client-facing resolver.

- `dns-resolv-02.servers.home.lab`
  - IP address: `10.10.20.13`.
  - Role: client-facing resolver.

Client configuration validated on `ws01`:

- DNS servers: `10.10.20.12`, `10.10.20.13`.
- Search domain: `home.lab`.
- System resolver path: `127.0.0.53` through `systemd-resolved`.

## Network Placement

DNS services are placed in the Server zone.

- Zone: SRV.
- VLAN: 20.
- Subnet: `10.10.20.0/24`.
- Gateway: `10.10.20.1`.

Related infrastructure records also cover:

- MGMT / VLAN 10: Proxmox and infrastructure management records.
- ADMIN / VLAN 30: admin workstation records.
- CLIENT / VLAN 40: future client records.
- GST-IOT / VLAN 50: future guest/IoT records.
- DMZ / VLAN 60: future DMZ records.

## Forward Zones

Active internal forward zones:

- `home.lab`
- `mgmt.home.lab`
- `servers.home.lab`
- `admin.home.lab`
- `clients.home.lab`
- `guest.home.lab`
- `dmz.home.lab`

Notes:

- `home.lab` is the parent internal namespace.
- Subzones separate records by infrastructure/network purpose.
- Empty or low-node-count zones are acceptable when the network zone exists but hosts are not yet deployed.

## Reverse Zones

Active internal reverse zones:

- `10.10.10.in-addr.arpa` — MGMT / VLAN 10.
- `20.10.10.in-addr.arpa` — SRV / VLAN 20.
- `30.10.10.in-addr.arpa` — ADMIN / VLAN 30.
- `40.10.10.in-addr.arpa` — CLIENT / VLAN 40.
- `50.10.10.in-addr.arpa` — GST-IOT / VLAN 50.
- `60.10.10.in-addr.arpa` — DMZ / VLAN 60.

## Client Query Policy

Approved client behavior:

- Admin workstation `ws01` uses the resolver pair `10.10.20.12` and `10.10.20.13`.
- Client systems should use the resolver pair, not query arbitrary external resolvers directly.
- Authoritative servers should be managed as infrastructure services, not as general-purpose login hosts.

Operational rule:

- Approved internal clients should query the resolver pair.
- Resolver-to-authoritative behavior should remain documented and restricted by firewall policy where practical.
- Guest/IoT DNS access should be allowed only after a documented decision.

## Firewall Requirements

DNS query traffic:

- Approved VLANs to DNS resolver pair: UDP/53.
- Approved VLANs to DNS resolver pair: TCP/53 for large responses and fallback.

Authoritative DNS traffic:

- Secondary DNS must be able to transfer zones from the primary authoritative server.
- DNS zone transfer traffic should be limited to approved secondary servers.

Administrative traffic:

- SSH administration should be limited to the Admin zone.

Firewall documentation should be maintained in:

- `docs/network/current-firewall-policy.md`


## Implementation Milestones

Completed in Phase 6:

- [x] Select internal domain: `home.lab`.
- [x] Implement BIND9 authoritative DNS.
- [x] Configure primary authoritative zones.
- [x] Configure secondary authoritative zones.
- [x] Configure forward zones for `home.lab` and network subzones.
- [x] Configure reverse zones for VLAN subnets 10-60.
- [x] Configure resolver pair for client DNS.
- [x] Configure `ws01` to use `10.10.20.12` and `10.10.20.13`.
- [x] Validate forward lookups from `ws01`.
- [x] Validate reverse lookups from `ws01`.
- [x] Correct PTR trailing-dot issue in the SRV reverse zone.
- [x] Document validation evidence.

Deferred to later phases:

- [ ] DHCP-DDNS integration.
- [ ] Public DNS/DDNS.
- [ ] Reverse proxy records.
- [ ] TLS certificate automation.
- [ ] Monitoring checks for DNS health.
- [ ] Backup/restore test for DNS configuration.

## Validation Summary

Validated from `ws01` through the normal system resolver path:

```bash
dig pve01.mgmt.home.lab
dig pve02.mgmt.home.lab
dig dns-resolv-01.servers.home.lab
dig dns-resolv-02.servers.home.lab
dig -x 10.10.10.10
dig -x 10.10.10.11
dig -x 10.10.20.12
dig -x 10.10.20.13
```

Expected behavior:

- Internal forward records return the correct internal IP addresses.
- Internal reverse records return correct fully qualified domain names.
- Client lookups use `127.0.0.53`, proving integration with the normal `systemd-resolved` path on `ws01`.

## Operational Notes

- DNS record targets in reverse zone files must use trailing dots when the target is an absolute FQDN.
- Example correct PTR target:

```bind
12  IN  PTR  dns-resolv-01.servers.home.lab.
```

- Missing trailing dots cause BIND to append the current zone origin, creating incorrect names.

## Remaining Follow-up Items

- Add monitoring checks for resolver availability and known-record lookups during the monitoring phase.
- Add backup and restore procedure for BIND configuration and zone files during the backup phase.
- Review whether client VLANs should be forced to internal resolvers through firewall policy.
- Keep DNS records updated when new services are deployed.

