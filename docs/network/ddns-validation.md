# DDNS Validation

## Purpose

This document records validation evidence for Phase 9: DHCP-DDNS and external DDNS.

Current evidence covers internal DHCP-DDNS between Kea DHCP, Kea DHCP-DDNS / D2, and BIND9.

External DDNS is not validated yet and will be added after external DDNS is implemented.

## Phase

Phase: Phase 9 - DHCP-DDNS and external DDNS  
Current validation focus: Internal DHCP-DDNS  
Date: 2026-09-04  

## Validation summary

Internal DHCP-DDNS:

    PASS

External DDNS:

    Not validated yet

Overall Phase 9 status:

    In progress

## Internal DHCP-DDNS validation

### Test objective

Validate that a DHCP client on VLAN 30 Admin receives an IP address from Kea DHCP and is automatically registered in internal BIND9 DNS with matching forward and reverse records.

### Environment

Client hostname:

    michel

Client VLAN:

    VLAN 30 Admin

Client interface:

    eth0

Client IP address:

    10.10.30.101

DHCP service:
Kea DHCP

DDNS service:

    Kea DHCP-DDNS / D2

Authoritative DNS:

    BIND9 primary DNS

Primary DNS server:

    10.10.20.10

Forward zone:

    admin.home.lab

Reverse zone:

    30.10.10.in-addr.arpa

DDNS update key:

    kea-ddns

## DHCP lease validation

Command run on the DHCP client:

    ip a

Observed output:

    2: eth0@if12: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc noqueue state UP group default qlen 1000
        link/ether bc:24:11:28:fe:5a brd ff:ff:ff:ff:ff:ff link-netnsid 0
        inet 10.10.30.101/24 metric 1024 brd 10.10.30.255 scope global dynamic eth0
           valid_lft 86363sec preferred_lft 86363sec
        inet6 fe80::be24:11ff:fe28:fe5a/64 scope link proto kernel_ll
           valid_lft forever preferred_lft forever

Expected result:

    Client receives a dynamic address from the VLAN 30 Admin subnet.

Observed result:

    Client received 10.10.30.101/24 dynamically.

Result:

    PASS

## Forward DNS validation

Command:

    dig @10.10.20.10 michel.admin.home.lab A +short

Expected result:

    10.10.30.101

Observed result:

    10.10.30.101

Result:

    PASS

Conclusion:

    The forward A record was created or updated successfully.

Validated forward record:

    michel.admin.home.lab. A 10.10.30.101

## Reverse DNS validation

Command:

    nslookup 10.10.30.101

Expected result:

    10.10.30.101 resolves back to michel.admin.home.lab.

Observed result:

    101.30.10.10.in-addr.arpa  name = michel.admin.home.lab.

Result:

    PASS

Conclusion:

    The reverse PTR record was created or updated successfully.

Validated reverse record:

    101.30.10.10.in-addr.arpa. PTR michel.admin.home.lab.

## Internal DHCP-DDNS conclusion

Kea DHCP-DDNS successfully created matching forward and reverse DNS records for a DHCP client on VLAN 30 Admin.

This confirms that DHCP lease information can be automatically reflected in internal BIND9 DNS using TSIG-secured dynamic updates.

Internal DHCP-DDNS result:

    PASS

## Evidence proven

This validation proves:

- Kea DHCP can issue a dynamic lease on VLAN 30 Admin.
- Kea DHCP-DDNS / D2 can process DHCP name change requests.
- BIND9 can accept dynamic updates from Kea using the `kea-ddns` TSIG key.
- The `admin.home.lab` forward zone can receive dynamic A records.
- The `30.10.10.in-addr.arpa` reverse zone can receive dynamic PTR records.
- Forward and reverse DNS records match the DHCP lease.

## External DDNS validation

External DDNS status:

    Not validated yet

Reason:

    External DDNS provider, hostname, and update method have not been finalized in this evidence.

Planned future validation:

1. Select external DDNS provider.
2. Select external hostname.
3. Configure update method.
4. Detect current public WAN IP.
5. Update external DNS record.
6. Validate public DNS resolution.
7. Confirm public DNS result matches current WAN IP.
8. Document logs, result, and rollback steps.

Future external DDNS test commands:

    curl -4 ifconfig.me
    dig <external-hostname> A +short

Expected future result:

    External hostname resolves to current public WAN IP.

## Current acceptance criteria

- [x] DHCP client receives a dynamic VLAN 30 Admin address.
- [x] Forward DNS record is created for the DHCP client.
- [x] Reverse PTR record is created for the DHCP client.
- [x] BIND9 accepts dynamic updates from Kea DHCP-DDNS.
- [x] DNS records match the DHCP lease information.
- [ ] External DDNS provider is selected.
- [ ] External DDNS hostname is selected.
- [ ] External DDNS update method is implemented.
- [ ] External DDNS public hostname resolves to the current public WAN IP.
- [ ] External DDNS evidence is documented.

## Troubleshooting commands

Check DHCP client address:

    ip a

Check forward DNS:

    dig @10.10.20.10 michel.admin.home.lab A +short

Check reverse DNS:

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

## Portfolio summary

Validated internal DHCP-DDNS integration between Kea DHCP and BIND9. A VLAN 30 Admin DHCP client automatically received matching forward and reverse DNS records using TSIG-secured dynamic updates.

## Final status

Internal DHCP-DDNS:

    PASS

External DDNS:

    Planned / not yet validated

Phase 9:

    In progress

