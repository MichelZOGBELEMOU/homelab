# DNS Records

## Purpose

This document records active and planned DNS records for the internal `home.lab` homelab namespace.

Records should be updated when hosts or services are created, moved, renamed, or retired.

## Record Status Legend

- Active: record exists and has been tested.
- Planned: record is intended but not implemented yet.
- Deferred: record depends on a future phase or service.
- Needs verification: record may exist but has not been confirmed.
- Retired: record should no longer be used.

## Naming Rules

- Use lowercase names.
- Use predictable infrastructure names.
- Use subzones to reflect network/service purpose.
- Do not publish internal-only names to public DNS.
- Use fully qualified names with trailing dots in BIND zone files when the target is absolute.

Internal domain:

- `home.lab`

Active subzones:

- `mgmt.home.lab`
- `servers.home.lab`
- `admin.home.lab`
- `clients.home.lab`
- `guest.home.lab`
- `dmz.home.lab`

## Active Forward Records

### Management zone: `mgmt.home.lab`

- `pve01.mgmt.home.lab`
  - IP address: `10.10.10.10`
  - Purpose: Proxmox host 1 management.
  - Status: Active.
  - Validation: `dig pve01.mgmt.home.lab` returned `10.10.10.10`.

- `pve02.mgmt.home.lab`
  - IP address: `10.10.10.11`
  - Purpose: Proxmox host 2 management.
  - Status: Active.
  - Validation: `dig pve02.mgmt.home.lab` returned `10.10.10.11`.

### Server zone: `servers.home.lab`

- `dns-resolv-01.servers.home.lab`
  - IP address: `10.10.20.12`
  - Purpose: primary client-facing DNS resolver.
  - Status: Active.
  - Validation: `dig dns-resolv-01.servers.home.lab` returned `10.10.20.12`.

- `dns-resolv-02.servers.home.lab`
  - IP address: `10.10.20.13`
  - Purpose: secondary client-facing DNS resolver.
  - Status: Active.
  - Validation: `dig dns-resolv-02.servers.home.lab` returned `10.10.20.13`.

## DNS Infrastructure Roles

### Authoritative DNS

- `dns01.home.lab`
  - Role: primary authoritative DNS server.
  - Software: BIND9.
  - Zone type: primary.
  - Status: Active.

- `dns02.home.lab`
  - Role: secondary authoritative DNS server.
  - Software: BIND9.
  - Zone type: secondary.
  - Status: Active.

### Client Resolvers

- `dns-resolv-01.servers.home.lab`
  - IP address: `10.10.20.12`.
  - Role: resolver used by clients.
  - Status: Active.

- `dns-resolv-02.servers.home.lab`
  - IP address: `10.10.20.13`.
  - Role: resolver used by clients.
  - Status: Active.

## Active Reverse Records

### MGMT reverse zone: `10.10.10.in-addr.arpa`

- `10.10.10.10`
  - PTR target: `pve01.mgmt.home.lab.`
  - Status: Active.
  - Validation: `dig -x 10.10.10.10` returned `pve01.mgmt.home.lab.`.

- `10.10.10.11`
  - PTR target: `pve02.mgmt.home.lab.`
  - Status: Active.
  - Validation: `dig -x 10.10.10.11` returned `pve02.mgmt.home.lab.`.

### SRV reverse zone: `20.10.10.in-addr.arpa`

- `10.10.20.12`
  - PTR target: `dns-resolv-01.servers.home.lab.`
  - Status: Active.
  - Validation: `dig -x 10.10.20.12` returned `dns-resolv-01.servers.home.lab.`.

- `10.10.20.13`
  - PTR target: `dns-resolv-02.servers.home.lab.`
  - Status: Active.
  - Validation: `dig -x 10.10.20.13` returned `dns-resolv-02.servers.home.lab.`.

## Zone Inventory

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

## Planned or Deferred Records

Future service records should be added when those services are built and validated.

- Monitoring service records: Deferred.
- Logging service records: Deferred.
- Backup service records: Deferred.
- Zammad/service-desk records: Deferred.
- DHCP/DDNS-generated client records: Deferred.
- Reverse proxy and public-service records: Deferred.

## Validation Checklist

Completed validation from `ws01`:

- [x] `pve01.mgmt.home.lab` resolves to `10.10.10.10`.
- [x] `pve02.mgmt.home.lab` resolves to `10.10.10.11`.
- [x] `dns-resolv-01.servers.home.lab` resolves to `10.10.20.12`.
- [x] `dns-resolv-02.servers.home.lab` resolves to `10.10.20.13`.
- [x] `10.10.10.10` reverses to `pve01.mgmt.home.lab.`.
- [x] `10.10.10.11` reverses to `pve02.mgmt.home.lab.`.
- [x] `10.10.20.12` reverses to `dns-resolv-01.servers.home.lab.`.
- [x] `10.10.20.13` reverses to `dns-resolv-02.servers.home.lab.`.

## Maintenance Rules

When adding or changing records:

1. Update the correct forward zone.
2. Update the matching reverse zone if the address is static infrastructure.
3. Use a trailing dot for absolute PTR targets.
4. Increment the zone serial.
5. Run `named-checkzone`.
6. Reload the zone with `rndc`.
7. Validate with `dig` from a client, not only from the DNS server.
8. Update this document with the new record and validation result.

