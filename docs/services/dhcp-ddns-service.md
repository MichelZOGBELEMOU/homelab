# DHCP-DDNS Service

## Purpose

This document describes the DHCP-DDNS service used in the homelab.

DHCP-DDNS connects DHCP lease assignment with internal DNS record management. When Kea DHCP assigns a lease, Kea DHCP-DDNS / D2 sends secure dynamic DNS updates to BIND9 so that DHCP clients can be resolved by hostname.

## Phase

Phase: Phase 9 - DHCP-DDNS and external DDNS  
Service status: Internal DHCP-DDNS active for VLAN 30 Admin  
Validation status: PASS for tested Admin VLAN client  

## Service summary

Service name:

    DHCP-DDNS

Primary software components:

- Kea DHCP
- Kea DHCP-DDNS / D2
- BIND9 authoritative DNS

Service purpose:

- Automatically register DHCP clients in internal DNS.
- Create forward A records for DHCP clients.
- Create reverse PTR records for DHCP clients.
- Reduce manual DNS record management.
- Improve troubleshooting visibility by mapping IP addresses to hostnames.

## Service dependencies

DHCP-DDNS depends on:

- Working VLAN addressing
- Working Kea DHCP service
- Working BIND9 authoritative DNS service
- Existing forward DNS zones
- Existing reverse DNS zones
- Valid TSIG key configuration
- Network reachability from Kea DHCP-DDNS to BIND9

## Current service endpoints

Kea DHCP-DDNS listener:

    127.0.0.1:53001

BIND9 primary authoritative DNS server:

    10.10.20.10:53

Validated forward zone:

    admin.home.lab

Validated reverse zone:

    30.10.10.in-addr.arpa

## Current TSIG keys

DDNS update key:

    kea-ddns

Zone transfer key:

    dns-transfer

Security note:

Do not commit TSIG secrets to the repository. Repository documentation should record key names and purpose only, not key values.

## Current validated client

Client hostname:

    michel

Client VLAN:

    VLAN 30 Admin

Client IP address:

    10.10.30.101

Created forward record:

    michel.admin.home.lab. A 10.10.30.101

Created reverse record:

    101.30.10.10.in-addr.arpa. PTR michel.admin.home.lab.

## Service workflow

1. A client connects to the Admin VLAN.
2. Kea DHCP assigns an IP address.
3. Kea DHCP receives or determines the client hostname.
4. Kea DHCP sends a name change request to Kea DHCP-DDNS / D2.
5. Kea DHCP-DDNS signs the DNS update using the `kea-ddns` TSIG key.
6. BIND9 validates the TSIG key and update policy.
7. BIND9 updates the forward zone.
8. BIND9 updates the reverse zone.
9. Internal DNS queries return the DHCP client hostname and IP address.

## Current BIND9 update policy

The Admin forward zone allows updates from the Kea DDNS key:

    update-policy {
      grant kea-ddns zonesub ANY;
    };

The VLAN 30 reverse zone allows updates from the Kea DDNS key:

    update-policy {
      grant kea-ddns zonesub ANY;
    };

## Service validation

Validation evidence is stored in:

- `docs/network/ddns-validation.md`

Current validation result:

    PASS

Validated tests:

- DHCP client received a dynamic lease.
- Forward A record resolved correctly.
- Reverse PTR record resolved correctly.
- BIND9 accepted dynamic updates from Kea DHCP-DDNS.

## Operations commands

Check DHCP client address:

    ip a

Check forward DNS record:

    dig @10.10.20.10 michel.admin.home.lab A +short

Check reverse DNS record:

    dig @10.10.20.10 -x 10.10.30.101 +short

Check Kea DHCP service:

    systemctl status kea-dhcp4-server

Check Kea DHCP-DDNS service:

    systemctl status kea-dhcp-ddns-server

Check BIND9 service:

    systemctl status bind9

Check recent Kea DHCP-DDNS logs:

    journalctl -u kea-dhcp-ddns-server --since "1 hour ago"

Check recent BIND9 logs:

    journalctl -u bind9 --since "1 hour ago"

## Troubleshooting notes

If forward DNS fails:

- Confirm the client received a DHCP lease.
- Confirm the hostname is present in the DHCP lease information.
- Check Kea DHCP-DDNS logs.
- Check BIND9 logs for refused updates or TSIG failures.
- Confirm the forward zone allows updates from the `kea-ddns` key.
- Confirm the Kea DHCP-DDNS forward domain matches the BIND9 zone.

If reverse DNS fails:

- Confirm the reverse zone exists in BIND9.
- Confirm the reverse zone name matches the client subnet.
- Confirm Kea DHCP-DDNS reverse domain configuration is correct.
- Confirm the PTR target has the correct fully qualified hostname.
- Check for BIND9 update-policy or TSIG errors.

If updates are refused:

- Confirm the TSIG key name matches on Kea and BIND9.
- Confirm the TSIG secret file exists and is readable by Kea.
- Confirm the BIND9 update-policy references the correct key.
- Confirm BIND9 is receiving updates on the expected server.

## Risks and controls

Risk: Stale DNS records after lease changes  
Control: Test lease change and hostname change behavior later.

Risk: Unauthorized DNS updates  
Control: Use TSIG-secured updates and restrict BIND9 update policies.

Risk: Dynamic records overwrite static records  
Control: Keep important infrastructure records static and review update-policy scope.

Risk: Documentation exposes secrets  
Control: Do not commit TSIG secrets, tokens, or private key material.

## Current service status

Internal DHCP-DDNS service:

    Working for VLAN 30 Admin

External DDNS:

    Not part of this service document

Overall status:

    Active and partially validated
