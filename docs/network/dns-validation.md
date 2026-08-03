# DNS Validation Report

## Purpose

This document records the validation evidence for Phase 6 — DNS service.

The goal is to prove that internal DNS works from the client perspective, not only from the DNS servers themselves.

## Validation Date

- Date observed: 2026-08-03.
- Client used for validation: `ws01`.
- Client resolver stack: `systemd-resolved` using local stub resolver `127.0.0.53`.

## Client Resolver Configuration

Command:

    resolvectl

Observed result:

    Link 4 (bridge0.30)
        Current Scopes: DNS
             Protocols: +DefaultRoute -LLMNR -mDNS -DNSOverTLS DNSSEC=no/unsupported
    Current DNS Server: 10.10.20.12
           DNS Servers: 10.10.20.12 10.10.20.13
            DNS Domain: home.lab
         Default Route: yes

Interpretation:

- `ws01` is using the internal DNS resolver pair.
- The configured DNS servers are `10.10.20.12` and `10.10.20.13`.
- The search domain is `home.lab`.
- DNS queries from normal tools use the system resolver path.

## Forward Lookup Tests

### `pve01.mgmt.home.lab`

Command:

    dig pve01.mgmt.home.lab

Result:

    pve01.mgmt.home.lab.  3600  IN  A  10.10.10.10
    SERVER: 127.0.0.53#53

Status: Passed.

### `pve02.mgmt.home.lab`

Command:

    dig pve02.mgmt.home.lab

Result:

    pve02.mgmt.home.lab.  3600  IN  A  10.10.10.11
    SERVER: 127.0.0.53#53

Status: Passed.

### `dns-resolv-01.servers.home.lab`

Command:

    dig dns-resolv-01.servers.home.lab

Result:

    dns-resolv-01.servers.home.lab.  3600  IN  A  10.10.20.12
    SERVER: 127.0.0.53#53

Status: Passed.

### `dns-resolv-02.servers.home.lab`

Command:

    dig dns-resolv-02.servers.home.lab

Result:

    dns-resolv-02.servers.home.lab.  3600  IN  A  10.10.20.13
    SERVER: 127.0.0.53#53

Status: Passed.

## Reverse Lookup Tests

### `10.10.10.10`

Command:

    dig -x 10.10.10.10

Result:

    10.10.10.10.in-addr.arpa.  IN  PTR  pve01.mgmt.home.lab.
    SERVER: 127.0.0.53#53

Status: Passed.

### `10.10.10.11`

Command:

    dig -x 10.10.10.11

Result:

    11.10.10.10.in-addr.arpa.  IN  PTR  pve02.mgmt.home.lab.
    SERVER: 127.0.0.53#53

Status: Passed.

### `10.10.20.12`

Command:

    dig -x 10.10.20.12

Result:

    12.20.10.10.in-addr.arpa.  3600  IN  PTR  dns-resolv-01.servers.home.lab.
    SERVER: 127.0.0.53#53

Status: Passed.

### `10.10.20.13`

Command:

    dig -x 10.10.20.13

Result:

    13.20.10.10.in-addr.arpa.  3600  IN  PTR  dns-resolv-02.servers.home.lab.
    SERVER: 127.0.0.53#53

Status: Passed.

## Authoritative Zone Status Evidence

Primary authoritative DNS showed primary zones loaded for internal forward and reverse zones.

Examples from `dns01`:

    sudo rndc zonestatus home.lab
    sudo rndc zonestatus mgmt.home.lab
    sudo rndc zonestatus servers.home.lab
    sudo rndc zonestatus clients.home.lab
    sudo rndc zonestatus guest.home.lab
    sudo rndc zonestatus dmz.home.lab
    sudo rndc zonestatus 10.10.10.in-addr.arpa
    sudo rndc zonestatus 20.10.10.in-addr.arpa
    sudo rndc zonestatus 30.10.10.in-addr.arpa
    sudo rndc zonestatus 40.10.10.in-addr.arpa
    sudo rndc zonestatus 50.10.10.in-addr.arpa
    sudo rndc zonestatus 60.10.10.in-addr.arpa

Secondary authoritative DNS showed secondary zones loaded from cache.

Examples from `dns02`:

    sudo rndc zonestatus home.lab
    sudo rndc zonestatus servers.home.lab
    sudo rndc zonestatus admin.home.lab
    sudo rndc zonestatus clients.home.lab
    sudo rndc zonestatus guest.home.lab
    sudo rndc zonestatus dmz.home.lab
    sudo rndc zonestatus 10.10.10.in-addr.arpa
    sudo rndc zonestatus 20.10.10.in-addr.arpa
    sudo rndc zonestatus 30.10.10.in-addr.arpa
    sudo rndc zonestatus 40.10.10.in-addr.arpa
    sudo rndc zonestatus 50.10.10.in-addr.arpa
    sudo rndc zonestatus 60.10.10.in-addr.arpa

Validation meaning:

- Primary authoritative zones are loaded.
- Secondary authoritative zones are loaded.
- Reverse zones exist for VLANs 10-60.
- Secondary zone files are stored under `/var/cache/bind/secondaries/`.

## Troubleshooting Finding

During final validation, two PTR records initially returned incorrect values:

    dns-resolv-01.servers.home.lab.20.10.10.in-addr.arpa.
    dns-resolv-02.servers.home.lab.20.10.10.in-addr.arpa.

Root cause:

- The PTR targets in the reverse zone were missing trailing dots.
- BIND treated the names as relative and appended the reverse-zone origin.

Corrected records:

    12  IN  PTR  dns-resolv-01.servers.home.lab.
    13  IN  PTR  dns-resolv-02.servers.home.lab.

Validation after correction:

    12.20.10.10.in-addr.arpa.  3600  IN  PTR  dns-resolv-01.servers.home.lab.
    13.20.10.10.in-addr.arpa.  3600  IN  PTR  dns-resolv-02.servers.home.lab.

Status: Fixed and verified.

## Acceptance Criteria

- [x] `ws01` uses internal DNS resolvers.
- [x] `ws01` has `home.lab` as DNS domain.
- [x] Forward lookup for management hosts works.
- [x] Forward lookup for resolver hosts works.
- [x] Reverse lookup for management hosts works.
- [x] Reverse lookup for resolver hosts works.
- [x] Authoritative primary zones are loaded.
- [x] Authoritative secondary zones are loaded.
- [x] DNS troubleshooting issue was identified, corrected, and documented.
## Firewall Validation for DNS

### Purpose

This validation confirms that inter-VLAN firewall policy allows internal DNS queries to the approved resolver pair while preserving default-deny behavior between zones.

### Validated resolver targets

Approved DNS resolvers:

- `10.10.20.12` — `dns-resolv-01.servers.home.lab`
- `10.10.20.13` — `dns-resolv-02.servers.home.lab`

Firewall group:

- `DNS-RESOLVER`

### MGMT to SRV DNS validation

Ruleset:
- `MGMT-TO-SRV`

Validated rule:

- Rule number: `60`
- Action: accept
- Protocol: TCP/UDP
- Destination port: `53`
- Destination group: `DNS-RESOLVER`

Observed match condition:

    meta l4proto { tcp, udp } th dport 53 ip daddr @A_DNS-RESOLVER

Observed counters:

- Packets: `108`
- Bytes: `7184`

Validation result:

- Passed.
- MGMT-to-SRV DNS traffic is restricted to the approved resolver group.

### ADMIN to SRV DNS validation

Ruleset:

- `ADMIN-TO-SRV`

Validated rule:

- Rule number: `90`
- Action: accept
- Protocol: TCP/UDP
- Destination port: `53`

Observed match condition:

    meta l4proto { tcp, udp } th dport 53

Observed counters:

- Packets: `1946`
- Bytes: `150547`

Validation result:

- Passed.
- ADMIN clients can reach DNS on TCP/UDP port `53` in the SRV VLAN.

Hardening note:

- This rule currently allows DNS to the SRV VLAN generally.
- Future improvement: restrict the destination to firewall group `DNS-RESOLVER`.

### Return traffic validation

Rulesets:

- `SRV-TO-MGMT`
- `SRV-TO-ADMIN`

Observed policy:

- Established traffic is accepted.
- Related traffic is accepted.
- Invalid traffic is dropped.
- Default policy is drop.

Validation result:

- Passed.
- DNS replies can return to MGMT and ADMIN clients.
- SRV hosts are not allowed to initiate arbitrary new traffic into MGMT or ADMIN by default.

### DNS firewall validation result

Status: Passed.

The firewall policy supports the internal DNS service design:

- Approved resolver pair is reachable.
- DNS queries match expected firewall rules.
- Return traffic is limited to established/related flows.
- Inter-VLAN default drop behavior remains in place.

### Follow-up item

Restrict `ADMIN-TO-SRV` DNS access to the `DNS-RESOLVER` destination group to match the stricter MGMT DNS policy.

## Final Result

Phase 6 DNS core validation passed.

The homelab now has internal DNS evidence covering authoritative DNS, secondary zones, resolver configuration, forward records, reverse records, and client-side validation.

