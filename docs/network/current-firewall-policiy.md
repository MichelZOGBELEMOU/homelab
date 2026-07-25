# Current Firewall Policy

## Purpose

This document records the current baseline firewall policy for the homelab network segmentation phase.

The goal is to define which network zones are allowed to communicate, which traffic is denied by default, and how future firewall rules will be added safely as services are installed.

This policy supports Phase 5: IP plan, naming, network zones, VLAN segmentation, and firewall baseline.

---

## Firewall Policy Status

Status: Baseline policy in progress  
Scope: VyOS router/firewall and VLAN-based homelab network  
Last reviewed: 2026-07-26  

At this stage, the firewall policy focuses on:

- Separating management, server, admin, client, guest/IoT, and DMZ networks
- Allowing only required administrative access
- Denying unnecessary inter-VLAN traffic
- Deferring service-specific rules until the related service is installed and tested

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

- Internal DNS
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
| CLIENT VLAN firewall tests | Validate when client host exists | DEFERRED |
| SRV VLAN firewall tests | Validate when server services exist | DEFERRED |
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

```text
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
```


---

## Current Policy Summary

The current firewall baseline is:

- ADMIN can manage MGMT infrastructure.
- MGMT systems can reach the Internet for updates and required infrastructure traffic.
- CLIENT, GST-IOT, and DMZ cannot manage infrastructure.
- Inter-VLAN traffic is denied unless explicitly required.
- Service-specific firewall rules are deferred until the related service exists.
- Every future rule must be documented and validated.

This creates a safe starting point for building the rest of the homelab services without allowing unnecessary network access by default.

