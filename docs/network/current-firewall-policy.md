# Current Firewall Policy

## Purpose

This document records the current baseline firewall policy for the homelab network segmentation phase.

The goal is to define which network zones are allowed to communicate, which traffic is denied by default, and how service-specific firewall rules are added safely as infrastructure services are installed.

This policy supports:

- Phase 5: IP plan, naming, network zones, VLAN segmentation, and firewall baseline
- Phase 6: Internal DNS service access and validation

---

## Firewall Policy Status

Status: Baseline policy active with DNS service rules added  
Scope: VyOS router/firewall and VLAN-based homelab network  
Last reviewed: 2026-08-03  

The firewall policy focuses on:

- Separating management, server, admin, client, guest/IoT, and DMZ networks
- Allowing only required administrative and service traffic
- Denying unnecessary inter-VLAN traffic
- Adding service-specific rules only after the related service exists and has been tested
- Recording each new firewall rule with source, destination, port, protocol, reason, validation result, and follow-up hardening notes

---

## Network Zones

| Zone | VLAN | Subnet | Gateway | Purpose |
|---|---:|---|---|---|
| MGMT | 10 | 10.10.10.0/24 | 10.10.10.1 | Infrastructure management network |
| SRV | 20 | 10.10.20.0/24 | 10.10.20.1 | Internal server/services network |
| ADMIN | 30 | 10.10.30.0/24 | 10.10.30.1 | Administrator workstation network |
| CLIENT | 40 | 10.10.40.0/24 | 10.10.40.1 | Normal client/user devices |
| GST-IOT | 50 | 10.10.50.0/24 | 10.10.50.1 | Guest and IoT devices |
| DMZ | 60 | 10.10.60.0/24 | 10.10.60.1 | Externally exposed or semi-isolated services |

Admin and Client are separate zones. Admin is for trusted administration workstations. Client is for normal user/device traffic.

---

## Default Security Model

The default policy is:

- Allow required traffic only
- Deny unnecessary inter-VLAN traffic
- Do not allow Client, Guest/IoT, or DMZ networks to manage infrastructure
- Add service-specific rules only after the service exists and has a documented reason
- Record every new firewall rule with source, destination, port, protocol, reason, validation result, and rollback or hardening notes
- Prefer destination groups for service rules where possible

---

## Baseline Allow Rules

### ADMIN to MGMT

Allowed.

Purpose:

- Allow administrator workstation access to infrastructure devices
- Permit management of Proxmox, VyOS, switch, and other infrastructure systems

Expected traffic:

- ICMP/ping for basic reachability testing
- SSH to approved management hosts
- Web UI access to approved management hosts where required

Example allowed paths:

| Source Zone | Destination Zone | Traffic | Reason |
|---|---|---|---|
| ADMIN | MGMT | ICMP | Reachability testing |
| ADMIN | MGMT | SSH | Infrastructure administration |
| ADMIN | MGMT | HTTPS/Web UI | Management interfaces when needed |

---

### ADMIN to Internet

Allowed.

Purpose:

- Allow admin workstation to reach documentation, package repositories, GitHub, and other administrative resources

Expected traffic:

- DNS
- HTTP/HTTPS
- Package manager traffic

---

### MGMT to Internet

Allowed with caution.

Purpose:

- Allow infrastructure systems to update packages and reach required external resources

Expected traffic:

- DNS
- NTP
- HTTP/HTTPS for updates

Notes:

- This should not become unrestricted outbound access for unnecessary services.
- Future monitoring should show which management systems are making outbound connections.

---

## DNS Service Firewall Rules

### Purpose

These rules allow approved internal DNS traffic between VLANs while keeping default-deny firewall behavior between security zones.

The internal DNS resolver pair is located in the SRV VLAN:

- `10.10.20.12` — `dns-resolv-01.servers.home.lab`
- `10.10.20.13` — `dns-resolv-02.servers.home.lab`

Required DNS protocols:

- UDP `53` for normal DNS queries
- TCP `53` for large responses, retries, and fallback behavior

---

### DNS Resolver Firewall Group

Firewall group:

- Name: `DNS-RESOLVER`
- Type: address group

Members:

- `10.10.20.12`
- `10.10.20.13`

Purpose:

- Defines the approved internal DNS resolver targets.
- Allows firewall rules to reference DNS resolvers by group.
- Avoids repeating individual resolver IP addresses in multiple rules.
- Makes future resolver changes easier to maintain.

Observed reference:

- `ipv4-name-MGMT-TO-SRV-60`

Status:

- Active.

---

### MGMT to SRV DNS Rule

Ruleset:

- `MGMT-TO-SRV`

Rule details:

- Rule number: `60`
- Action: accept
- Protocol: TCP/UDP
- Destination port: `53`
- Destination group: `DNS-RESOLVER`

Observed condition:

    meta l4proto { tcp, udp } th dport 53 ip daddr @A_DNS-RESOLVER

Purpose:

- Allows management VLAN hosts to query the approved internal DNS resolver pair.
- Restricts MGMT DNS traffic to known resolver IPs only.
- Preserves default-deny behavior for other MGMT-to-SRV traffic.

Observed counters:

- Packets: `108`
- Bytes: `7184`

Validation status:

- Passed.

Operational note:

- This is the preferred pattern for service-specific firewall rules because the destination is restricted to the documented resolver group.

---

### SRV to MGMT Return Policy

Ruleset:

- `SRV-TO-MGMT`

Relevant behavior:

- Rule `10`: accept established traffic
- Rule `20`: accept related traffic
- Rule `30`: drop invalid traffic
- Default action: drop

Observed counters:

- Established packets: `120`
- Established bytes: `13737`
- Default drop packets: `0`
- Default drop bytes: `0`

Purpose:

- Allows DNS replies back to MGMT clients.
- Prevents SRV hosts from initiating arbitrary new connections into the MGMT VLAN.

Validation status:

- Passed.

---

### ADMIN to SRV DNS Rule

Ruleset:

- `ADMIN-TO-SRV`

Rule details:

- Rule number: `90`
- Action: accept
- Protocol: TCP/UDP
- Destination port: `53`

Observed condition:

    meta l4proto { tcp, udp } th dport 53

Purpose:

- Allows ADMIN VLAN clients, including `ws01`, to query DNS services in the SRV VLAN.
- Supports normal client-side DNS validation through `systemd-resolved`.

Observed counters:

- Packets: `1946`
- Bytes: `150547`

Validation status:

- Passed with follow-up hardening recommended.

Hardening note:

- Current rule allows DNS to TCP/UDP port `53` in the SRV VLAN generally.
- Future improvement: restrict this rule to destination group `DNS-RESOLVER`, matching the stricter `MGMT-TO-SRV` DNS rule.

Recommended future target condition:

    meta l4proto { tcp, udp } th dport 53 ip daddr @A_DNS-RESOLVER

---

### SRV to ADMIN Return Policy

Ruleset:

- `SRV-TO-ADMIN`

Relevant behavior:

- Rule `10`: accept established traffic
- Rule `20`: accept related traffic
- Rule `25`: drop invalid traffic
- Default action: drop

Observed counters:

- Established packets: `98771`
- Established bytes: `9052875`
- Default drop packets: `0`
- Default drop bytes: `0`

Purpose:

- Allows DNS replies back to ADMIN clients.
- Prevents SRV hosts from initiating arbitrary new connections into the ADMIN VLAN.

Validation status:

- Passed.

---

### DNS Firewall Validation Summary

The firewall policy supports the Phase 6 internal DNS design.

Validated behavior:

- MGMT clients can query only the approved DNS resolver group.
- ADMIN clients can query DNS in the SRV VLAN.
- SRV return traffic to MGMT and ADMIN is limited to established/related flows.
- Default drop remains active on inter-VLAN rulesets.
- DNS firewall counters show traffic matching the intended rules.

Validation evidence:

| Source Zone | Destination Zone | Ruleset | Rule | Protocol/Port | Destination Restriction | Status |
|---|---|---|---:|---|---|---|
| MGMT | SRV | `MGMT-TO-SRV` | 60 | TCP/UDP 53 | `DNS-RESOLVER` group | PASS |
| SRV | MGMT | `SRV-TO-MGMT` | 10/20 | established/related | return traffic only | PASS |
| ADMIN | SRV | `ADMIN-TO-SRV` | 90 | TCP/UDP 53 | SRV VLAN DNS access | PASS |
| SRV | ADMIN | `SRV-TO-ADMIN` | 10/20 | established/related | return traffic only | PASS |

Result:

- DNS firewall validation passed.

Follow-up hardening:

- Restrict `ADMIN-TO-SRV` rule `90` to destination group `DNS-RESOLVER`.
- Re-test DNS from `ws01` after the rule is narrowed.
- Confirm firewall counters increase on the updated ADMIN DNS rule.
- Update this document and `docs/network/dns-validation.md` after hardening.

---

## Baseline Deny Rules

### CLIENT to MGMT

Denied by default.

Reason:

- Normal client devices should not manage infrastructure systems.
- This reduces risk if a client device is compromised.

---

### GST-IOT to MGMT

Denied by default.

Reason:

- Guest and IoT devices are low-trust.
- They must not access infrastructure management interfaces.

---

### GST-IOT to SRV

Denied by default unless a specific service requires it.

Reason:

- Guest and IoT devices should not freely access internal servers.

---

### DMZ to MGMT

Denied by default.

Reason:

- DMZ systems may be exposed to external networks.
- DMZ hosts must not manage internal infrastructure.

---

### DMZ to SRV

Denied by default unless explicitly required.

Reason:

- DMZ services should not freely access internal server networks.
- Any required backend access must be documented as a specific service rule.

---

### CLIENT to ADMIN

Denied by default.

Reason:

- Client devices should not initiate access to administrator workstations.

---

### ADMIN and CLIENT Separation

Admin and Client are separate zones.

Policy:

- ADMIN may be allowed to manage infrastructure.
- CLIENT should be treated as normal user/device traffic.
- CLIENT should not inherit ADMIN privileges.

Reason:

- This models real-world separation between administrative access and ordinary endpoint access.

---

## Deferred Service-Specific Rules

The following rules are not added yet because the related services are not fully defined or installed.

Future rules may be needed for:

- DHCP/DHCP relay
- NTP
- Reverse proxy
- Monitoring
- Logging
- Backup services
- Identity/access services
- Zammad/helpdesk
- Mail services
- Public-facing DMZ services

Rules must not be added just because they may be useful later.

Each future rule must include:

- Source zone
- Destination zone or host
- Destination port/protocol
- Service name
- Business/operations reason
- Validation command
- Test result
- Date added
- Rollback or hardening note

---

## Current Validation Evidence

| Test | Expected Result | Status |
|---|---|---|
| ADMIN to MGMT gateway ping | Pass | PASS |
| ADMIN to Proxmox/MGMT host ping | Pass | PASS |
| ADMIN to pve01 SSH | Pass | PASS |
| ADMIN DNS resolution | Pass | PASS |
| ADMIN Internet access | Pass | PASS |
| MGMT host DNS resolution | Pass | PASS |
| MGMT host Internet access | Pass | PASS |
| MGMT to SRV DNS resolver group | TCP/UDP 53 allowed only to approved resolver group | PASS |
| ADMIN to SRV DNS | TCP/UDP 53 allowed for DNS resolution | PASS |
| SRV to MGMT return traffic | Established/related return traffic only | PASS |
| SRV to ADMIN return traffic | Established/related return traffic only | PASS |
| CLIENT VLAN firewall tests | Validate when client host exists | DEFERRED |
| GST-IOT VLAN firewall tests | Validate when guest/IoT host exists | DEFERRED |
| DMZ VLAN firewall tests | Validate when DMZ service exists | DEFERRED |

Deferred tests are acceptable at this stage because not every VLAN has active hosts or services yet.

---

## Rule Change Process

Before adding a new firewall rule, answer these questions:

1. What service requires this rule?
2. Which source zone needs access?
3. Which destination zone or host is being accessed?
4. Which port and protocol are required?
5. Is this temporary or permanent?
6. What command proves the rule works?
7. What command proves unnecessary traffic is still blocked?
8. Was the result documented?
9. Is the destination restricted as narrowly as possible?
10. Is there a rollback or hardening note?

Firewall rules should be added in small batches and tested immediately.

---

## Firewall Rule Documentation Template

Use this template for every future rule:

    Rule ID:
    Date:
    Source zone:
    Source host/subnet:
    Destination zone:
    Destination host/subnet:
    Port/protocol:
    Service:
    Reason:
    Validation command:
    Expected result:
    Actual result:
    Status:
    Rollback plan:
    Follow-up hardening:

---

## Current Policy Summary

The current firewall baseline is:

- ADMIN can manage MGMT infrastructure.
- MGMT systems can reach the Internet for updates and required infrastructure traffic.
- MGMT can query approved DNS resolvers in the SRV VLAN through the `DNS-RESOLVER` group.
- ADMIN can query DNS in the SRV VLAN.
- SRV return traffic to MGMT and ADMIN is limited to established/related flows.
- CLIENT, GST-IOT, and DMZ cannot manage infrastructure.
- Inter-VLAN traffic is denied unless explicitly required.
- Service-specific firewall rules must be documented and validated.
- Broad service rules should be narrowed with destination groups when possible.

This creates a safe starting point for building the rest of the homelab services without allowing unnecessary network access by default.

