# Current Firewall Policy

## Purpose

This document records the current baseline firewall policy for the homelab network segmentation phase.

The goal is to define which network zones are allowed to communicate, which traffic is denied by default, and how future firewall rules will be added safely as services are installed.

This policy supports Phase 5: IP plan, naming, network zones, VLAN segmentation, and firewall baseline. It also records the first service-specific firewall validation required for Phase 6: DNS service.

---

## Firewall Policy Status

Status: Baseline policy in progress  
Scope: VyOS router/firewall and VLAN-based homelab network  
Last reviewed: 2026-07-30  

At this stage, the firewall policy focuses on:

- Separating management, server, admin, client, guest/IoT, and DMZ networks
- Allowing only required administrative access
- Denying unnecessary inter-VLAN traffic
- Allowing limited service-specific traffic only when there is a documented reason
- Validating rules with real tests before marking them complete

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
- Do not allow servers to freely initiate connections to admin workstations
- Do not allow unsolicited WAN traffic into internal server networks
- Add service-specific rules only after the service exists and has a documented reason
- Record every new firewall rule with source, destination, port, protocol, reason, and validation result

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

### ADMIN to SRV

Allowed with specific service rules.

Purpose:

- Allow trusted administrator systems to manage and validate approved server systems.
- Support Phase 6 DNS deployment and testing.
- Allow administration from the ADMIN zone without giving servers broad access back to admin devices.

Expected traffic:

- ICMP for reachability and troubleshooting
- SSH to approved server hosts
- DNS queries to approved internal DNS servers
- HTTP/HTTPS for future approved server web interfaces when required

Current allowed traffic:

| Source Zone | Destination Zone | Traffic | Reason | Status |
|---|---|---|---|---|
| ADMIN | SRV | ICMP echo-request | Reachability testing | Active / validated |
| ADMIN | SRV | ICMP destination-unreachable | Network troubleshooting | Active / validated |
| ADMIN | SRV | ICMP time-exceeded | Network troubleshooting | Active / validated |
| ADMIN | SRV | TCP/22 | SSH administration | Active / validated |
| ADMIN | SRV | TCP/80 | Future HTTP service access when needed | Active |
| ADMIN | SRV | TCP/443 | Future HTTPS service access when needed | Active |
| ADMIN | SRV | TCP/UDP 53 | DNS queries to internal DNS service | Active / ready for DNS validation |

Default action:

- Drop all other ADMIN-to-SRV traffic.

Validation evidence:

- ADMIN to `dns01` ICMP passed.
- ADMIN to `dns01` SSH passed.
- Firewall counters showed traffic matching ADMIN-to-SRV rules.
- DNS rule is ready for validation after BIND9 is installed on `dns01`.

---

### SRV to ADMIN

Denied by default except return traffic.

Purpose:

- Prevent server systems from freely initiating connections to administrator workstations.
- Allow return traffic for connections started by ADMIN.

Current allowed traffic:

| Source Zone | Destination Zone | Traffic | Reason | Status |
|---|---|---|---|---|
| SRV | ADMIN | Established traffic | Return traffic for approved sessions | Active / validated |
| SRV | ADMIN | Related traffic | Related return traffic | Active / validated |

Current denied traffic:

| Source Zone | Destination Zone | Traffic | Reason | Status |
|---|---|---|---|---|
| SRV | ADMIN | Invalid traffic | Drop malformed or invalid packets | Active |
| SRV | ADMIN | New connections by default | Servers should not initiate access to admin devices | Active |

Default action:

- Drop all other SRV-to-ADMIN traffic.

Validation evidence:

- Return traffic from ADMIN-initiated SSH and ICMP tests worked.
- New SRV-to-ADMIN access is denied by default.

---

### SRV to Internet

Allowed with limited outbound rules.

Purpose:

- Allow server systems limited outbound access for package installation, security updates, DNS forwarding, and time synchronization.
- Support the Phase 6 `dns01` base install and later BIND9 operation.
- Avoid unrestricted outbound access from the server network.

Expected traffic:

- DNS for external name resolution
- HTTP/HTTPS for package repositories and software updates
- NTP for time synchronization
- ICMP for troubleshooting

Current allowed traffic:

| Source Zone | Destination Zone | Traffic | Reason | Status |
|---|---|---|---|---|
| SRV | WAN | ICMP echo-request | Internet reachability testing | Active / validated |
| SRV | WAN | ICMP destination-unreachable | Network troubleshooting | Active |
| SRV | WAN | ICMP time-exceeded | Network troubleshooting | Active |
| SRV | WAN | TCP/80 | Package repositories and updates | Active / validated |
| SRV | WAN | TCP/443 | Secure package repositories and downloads | Active / validated |
| SRV | WAN | TCP/UDP 53 | External DNS lookups and DNS forwarding | Active / validated |
| SRV | WAN | UDP/123 | NTP time synchronization | Active / validated |

Default action:

- Drop all other SRV-to-WAN traffic.

Validation evidence:

- `dns01` could ping `1.1.1.1`.
- `dns01` could resolve `debian.org`.
- `dns01` could reach the Internet from VLAN 20.
- Firewall counters showed matches for ICMP, DNS, HTTP, HTTPS, NTP, and established return traffic.

---

### WAN to SRV

Denied by default except return traffic.

Purpose:

- Prevent unsolicited inbound Internet traffic to internal server systems.
- Allow return traffic only for connections started by SRV systems.

Current allowed traffic:

| Source Zone | Destination Zone | Traffic | Reason | Status |
|---|---|---|---|---|
| WAN | SRV | Established traffic | Return traffic for SRV-initiated sessions | Active / validated |
| WAN | SRV | Related traffic | Related return traffic | Active / validated |

Current denied traffic:

| Source Zone | Destination Zone | Traffic | Reason | Status |
|---|---|---|---|---|
| WAN | SRV | Invalid traffic | Drop malformed or invalid packets | Active |
| WAN | SRV | New inbound connections by default | No public inbound access to SRV systems | Active |

Default action:

- Drop all other WAN-to-SRV traffic.

Validation evidence:

- Return traffic from SRV-initiated Internet access worked.
- No new inbound WAN-to-SRV service access is allowed.

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

- Internal DNS — partially active for Phase 6. ADMIN-to-SRV DNS access and SRV-to-WAN DNS access are allowed; BIND9 service validation is still pending.
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
| ADMIN to dns01 ping | Pass | PASS |
| ADMIN to dns01 SSH | Pass | PASS |
| dns01 to gateway ping | Pass | PASS |
| dns01 to Internet IP ping | Pass | PASS |
| dns01 external DNS resolution | Pass | PASS |
| SRV to WAN limited outbound policy | DNS, HTTP, HTTPS, NTP, and ICMP allowed | PASS |
| WAN to SRV default deny | No unsolicited inbound access allowed | PASS |
| SRV to ADMIN default deny | Only established/related return traffic allowed | PASS |
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

---

## Current Policy Summary

The current firewall baseline is:

- ADMIN can manage MGMT infrastructure.
- ADMIN can reach approved SRV systems for ICMP, SSH, DNS, and selected web access.
- SRV can reach WAN only for limited required outbound traffic: ICMP, DNS, NTP, HTTP, and HTTPS.
- SRV cannot initiate new connections to ADMIN by default.
- WAN cannot initiate new connections to SRV by default.
- MGMT systems can reach the Internet for updates and required infrastructure traffic.
- CLIENT, GST-IOT, and DMZ cannot manage infrastructure.
- Inter-VLAN traffic is denied unless explicitly required.
- Service-specific firewall rules must be documented and validated.

This creates a safe starting point for building the rest of the homelab services without allowing unnecessary network access by default.

