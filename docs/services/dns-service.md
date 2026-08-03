# DNS Service

## Service Summary

- Service name: Internal DNS.
- Internal domain: `home.lab`.
- Service model: BIND9 authoritative DNS plus redundant client resolvers.
- Network zone: SRV / VLAN 20.
- Status: Active and validated.
- Phase: Phase 6 — DNS service.
- Owner: Michel.

## Purpose

The DNS service provides internal hostname resolution for the homelab.

It allows administrators and future services to use stable names instead of memorizing IP addresses. It also creates a foundation for later DHCP-DDNS, reverse proxy, TLS, monitoring, logging, and service discovery work.

## Architecture

The DNS implementation is split into two roles.

### Authoritative DNS

- `dns01.home.lab`
  - Role: primary authoritative DNS server.
  - Software: BIND9.
  - Zone type: primary.
  - Zone files: `/var/lib/bind/`.

- `dns02.home.lab`
  - Role: secondary authoritative DNS server.
  - Software: BIND9.
  - Zone type: secondary.
  - Zone files: `/var/cache/bind/secondaries/`.

### Client Resolvers

- `dns-resolv-01.servers.home.lab`
  - IP address: `10.10.20.12`.
  - Role: client-facing resolver.

- `dns-resolv-02.servers.home.lab`
  - IP address: `10.10.20.13`.
  - Role: client-facing resolver.

Validated client:

- `ws01` uses DNS servers `10.10.20.12` and `10.10.20.13`.
- `ws01` search domain is `home.lab`.
- `ws01` uses the normal system resolver stub at `127.0.0.53`.

## Service Scope

In scope:

- Internal `home.lab` DNS.
- Network subzones.
- Forward records for infrastructure hosts.
- Reverse records for infrastructure hosts.
- Primary/secondary authoritative DNS.
- Client resolver pair.
- Client-side validation from `ws01`.

Out of scope:

- Public DNS.
- External DDNS.
- DHCP-DDNS integration.
- TLS certificates.
- Reverse proxy configuration.
- Public-facing services.

## Service Dependencies

Required dependencies:

- Server VLAN / VLAN 20 connectivity.
- Admin workstation access to the DNS resolver pair.
- BIND9 running on authoritative DNS servers.
- Zone transfer path from primary to secondary authoritative DNS.
- Correct resolver configuration on clients.

Related documentation:

- `docs/network/dns-plan.md`
- `docs/network/dns-records.md`
- `docs/network/dns-validation.md`
- `docs/network/ip-plan.md`
- `docs/network/network-zones.md`
- `docs/network/current-firewall-policiy.md`

## Active Zones

Forward zones:

- `home.lab`
- `mgmt.home.lab`
- `servers.home.lab`
- `admin.home.lab`
- `clients.home.lab`
- `guest.home.lab`
- `dmz.home.lab`

Reverse zones:

- `10.10.10.in-addr.arpa`
- `20.10.10.in-addr.arpa`
- `30.10.10.in-addr.arpa`
- `40.10.10.in-addr.arpa`
- `50.10.10.in-addr.arpa`
- `60.10.10.in-addr.arpa`

## Access Policy

Administrative access:

- Admin access to DNS servers should be limited to the Admin zone.
- SSH should be used for administration.

DNS query access:

- Approved clients should query the resolver pair.
- DNS traffic should use UDP/53 and TCP/53 as needed.
- Guest/IoT DNS access should remain denied unless explicitly documented later.

Zone-transfer access:

- Zone transfers should be allowed only from the primary authoritative DNS server to approved secondary authoritative DNS servers.

## Expected Ports

- UDP/53: standard DNS queries.
- TCP/53: large DNS responses, fallback, and DNS operations where required.
- TCP/22: SSH administration from approved admin systems.


## Firewall Requirements

### Required DNS access

The internal DNS service requires client networks to reach the DNS resolver pair in the SRV VLAN.

Approved resolver hosts:

- `dns-resolv-01.servers.home.lab` — `10.10.20.12`
- `dns-resolv-02.servers.home.lab` — `10.10.20.13`

Required ports:

- UDP `53` — standard DNS queries
- TCP `53` — large DNS responses and fallback queries

### Current firewall policy

MGMT to SRV:

- Ruleset: `MGMT-TO-SRV`
- Rule: `60`
- Access: TCP/UDP `53`
- Destination: `DNS-RESOLVER` firewall group
- Status: Passed

ADMIN to SRV:

- Ruleset: `ADMIN-TO-SRV`
- Rule: `90`
- Access: TCP/UDP `53`
- Destination: SRV VLAN DNS service
- Status: Passed
- Follow-up: restrict destination to `DNS-RESOLVER`

SRV return traffic:

- Rulesets:
  - `SRV-TO-MGMT`
  - `SRV-TO-ADMIN`
- Access: established/related only
- Default policy: drop
- Status: Passed

### Operational expectation

DNS should be reachable from approved client/admin networks only through the documented resolver pair.

The firewall should not allow broad inter-VLAN access simply because DNS is required. DNS access should be narrow, documented, and validated with packet counters and client lookup tests.

### Validation commands

From a client using the normal resolver path:

    dig pve01.mgmt.home.lab
    dig pve02.mgmt.home.lab
    dig -x 10.10.20.12
    dig -x 10.10.20.13

From the firewall:

    show firewall ipv4 name MGMT-TO-SRV
    show firewall ipv4 name ADMIN-TO-SRV
    show firewall ipv4 name SRV-TO-MGMT
    show firewall ipv4 name SRV-TO-ADMIN
    show firewall group DNS-RESOLVER

### Acceptance criteria

- DNS clients can resolve internal forward records.
- DNS clients can resolve internal reverse records.
- Firewall counters increase on expected DNS allow rules.
- DNS resolver targets are documented in the `DNS-RESOLVER` firewall group.
- Return traffic is handled by established/related rules.
- Default drop remains active between VLANs.
- Any broader temporary DNS rule is documented with a hardening follow-up.


## Availability Target

This is a homelab service, but DNS should be treated as core infrastructure.

Expected behavior:

- Resolver pair responds to approved clients.
- Internal forward records resolve consistently.
- Internal reverse records resolve consistently.
- Secondary authoritative zones remain loaded and refreshed.
- Failures are documented with commands, observations, root cause, and fix.

## Validation Evidence

Client resolver configuration on `ws01`:

    DNS Servers: 10.10.20.12 10.10.20.13
    DNS Domain: home.lab
    SERVER: 127.0.0.53#53

Forward lookups validated from `ws01`:

    dig pve01.mgmt.home.lab
dig pve02.mgmt.home.lab
    dig dns-resolv-01.servers.home.lab
    dig dns-resolv-02.servers.home.lab

Reverse lookups validated from `ws01`:

    dig -x 10.10.10.10
    dig -x 10.10.10.11
    dig -x 10.10.20.12
    dig -x 10.10.20.13

Confirmed results:

- `pve01.mgmt.home.lab` -> `10.10.10.10`.
- `pve02.mgmt.home.lab` -> `10.10.10.11`.
- `dns-resolv-01.servers.home.lab` -> `10.10.20.12`.
- `dns-resolv-02.servers.home.lab` -> `10.10.20.13`.
- `10.10.10.10` -> `pve01.mgmt.home.lab.`.
- `10.10.10.11` -> `pve02.mgmt.home.lab.`.
- `10.10.20.12` -> `dns-resolv-01.servers.home.lab.`.
- `10.10.20.13` -> `dns-resolv-02.servers.home.lab.`.

## Troubleshooting Note

During final validation, PTR records for the resolver hosts initially returned incorrect names with the reverse-zone origin appended:

    dns-resolv-01.servers.home.lab.20.10.10.in-addr.arpa.
    dns-resolv-02.servers.home.lab.20.10.10.in-addr.arpa.

Root cause:

- PTR targets in the `20.10.10.in-addr.arpa` reverse zone were missing trailing dots.
- BIND treated the targets as relative names and appended the reverse-zone origin.

Correct records:

    12  IN  PTR  dns-resolv-01.servers.home.lab.
    13  IN  PTR  dns-resolv-02.servers.home.lab.

Validation after correction:

    dig -x 10.10.20.12
    dig -x 10.10.20.13

The corrected answers returned the expected FQDNs.

## Backup and Recovery Notes

Backup requirements for the backup phase:

- BIND configuration files.
- Forward zone files.
- Reverse zone files.
- Resolver configuration notes.
- Validation commands and expected results.

Recovery expectation:

- A rebuilt DNS service should restore internal name resolution from documented configuration and zone files.

Backup implementation status:

- Deferred until backup phase.

## Monitoring Notes

Future monitoring should check:

- DNS resolver process/service is running.
- UDP/53 responds on both resolver IPs.
- TCP/53 responds when required.
- Known forward records resolve correctly.
- Known reverse records resolve correctly.
- Secondary zones are loaded and not expired.

Example future checks:

    dig @10.10.20.12 pve01.mgmt.home.lab
    dig @10.10.20.13 pve02.mgmt.home.lab
    dig @10.10.20.12 -x 10.10.20.12
    dig @10.10.20.13 -x 10.10.20.13

Monitoring implementation status:

- Deferred until monitoring phase.

## Implementation Checklist

- [x] Select internal domain.
- [x] Configure authoritative DNS.
- [x] Configure primary zones.
- [x] Configure secondary zones.
- [x] Configure forward zones.
- [x] Configure reverse zones.
- [x] Configure resolver pair.
- [x] Configure `ws01` client resolver settings.
- [x] Validate forward lookups.
- [x] Validate reverse lookups.
- [x] Troubleshoot and fix incorrect PTR target formatting.
- [x] Document validation evidence.

## Known Follow-up Items

- Add DNS monitoring checks in the monitoring phase.
- Add DNS backup/restore validation in the backup phase.
- Add DHCP-DDNS integration in a later DHCP/DDNS phase.
- Add reverse proxy and public DNS records only in later external access phases.

