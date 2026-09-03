# DHCP-DDNS Plan

## Purpose

This document describes the internal DHCP-DDNS design for the homelab.

DHCP-DDNS allows DHCP lease information from Kea DHCP to automatically create or update DNS records in BIND9. This reduces manual DNS record maintenance for DHCP clients and makes client hostnames easier to identify during troubleshooting.

## Phase

Phase: Phase 9 - DHCP-DDNS and external DDNS  
Status: Internal DHCP-DDNS partially implemented and validated  
External DDNS: Planned separately  

## Scope

This plan covers internal DHCP-DDNS only.

Included:

- Kea DHCP dynamic lease registration
- Kea DHCP-DDNS / D2 service
- BIND9 dynamic DNS updates
- TSIG-secured update authorization
- Forward DNS record updates
- Reverse PTR record updates
- VLAN-based internal DNS naming

Not included in this document:

- Public DNS
- External DDNS provider configuration
- Reverse proxy
- TLS certificates
- Public web access

External DDNS is documented separately in:

- `docs/network/external-ddns-plan.md`

## Design goal

The goal is for DHCP clients to automatically appear in internal DNS after receiving a DHCP lease.

Example:

A client named `michel` receives this DHCP lease:

    10.10.30.101

Kea DHCP-DDNS should create matching DNS records:

    michel.admin.home.lab. A 10.10.30.101
    101.30.10.10.in-addr.arpa. PTR michel.admin.home.lab.

## Internal DNS domain structure

The internal DNS domain is:

    home.lab

VLAN-specific subdomains are used for different network zones.

Current relevant zone for this validation:

    admin.home.lab

Current relevant reverse zone:

    30.10.10.in-addr.arpa

## DHCP-DDNS components

### Kea DHCP

Kea DHCP assigns client IP addresses and sends name change requests to Kea DHCP-DDNS / D2.

### Kea DHCP-DDNS / D2

Kea DHCP-DDNS receives name change requests from Kea DHCP and sends DNS update messages to BIND9.

Current local listener:

    127.0.0.1:53001

### BIND9 authoritative DNS

BIND9 stores the internal forward and reverse zones and accepts dynamic updates from Kea DHCP-DDNS.

Current primary authoritative DNS server:

    10.10.20.10

## Security model

Dynamic DNS updates are authorized using TSIG.

Current DDNS update key name:

    kea-ddns

BIND9 uses `update-policy` to allow the Kea DDNS key to update records inside the relevant zones.

Example policy:

    update-policy {
      grant kea-ddns zonesub ANY;
    };

Zone transfers use a separate transfer key:

    dns-transfer

## Current implemented zone coverage

Currently validated:

- Forward zone: `admin.home.lab`
- Reverse zone: `30.10.10.in-addr.arpa`
- VLAN: `VLAN 30 Admin`
- Client: `michel`
- Client IP: `10.10.30.101`

Planned or to be validated later:

- `mgmt.home.lab`
- `servers.home.lab`
- `clients.home.lab`
- `guest.home.lab`
- `dmz.home.lab`
- Reverse zones for other VLANs where DHCP-DDNS is required

## Current DHCP-DDNS behavior

Kea DHCP is configured to send DDNS updates.

Current behavior:

- Send DDNS updates: enabled
- Override client no-update requests: enabled
- Override client update requests: enabled
- Update on renew: disabled
- Qualifying suffix for Admin VLAN: `admin.home.lab.`

Relevant DHCP-DDNS behavior:

    "ddns-send-updates": true
    "ddns-override-no-update": true
    "ddns-override-client-update": true
    "ddns-update-on-renew": false
    "ddns-qualifying-suffix": "admin.home.lab."

## Current Kea DHCP-DDNS target

Forward DDNS domain:

    admin.home.lab.

Reverse DDNS domain:

    30.10.10.in-addr.arpa.

DNS update target:

    10.10.20.10 port 53

## Operational notes

DHCP-DDNS should be handled carefully because incorrect updates can create stale or misleading DNS records.

Important operational checks:

- Confirm the client hostname before relying on DNS records.
- Confirm forward and reverse records match the active DHCP lease.
- Confirm stale records are removed or replaced when leases change.
- Confirm BIND9 accepts updates only from authorized TSIG keys.
- Confirm dynamic records do not overwrite important static infrastructure records.

## Validation document

Validation evidence is recorded in:

- `docs/network/ddns-validation.md`

## Current status

Internal DHCP-DDNS for VLAN 30 Admin:

    PASS

External DDNS:

    Planned

Overall Phase 9:

    In progress

