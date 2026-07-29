# Hostname Standard

## Purpose

This document defines the hostname standard for the homelab.

The goal is to keep device and service names predictable, short, lowercase, and easy to use in DNS records, documentation, troubleshooting notes, and monitoring.

## DNS Domain

Internal DNS domain:

- `home.lab`

Hostname format:

- `<hostname>.home.lab`

Example:

- `dns01.home.lab`

## General Rules

Hostnames should be:

- lowercase
- short but meaningful
- predictable
- easy to sort when numbered
- based on role or stable device identity
- documented before being added to DNS

Avoid hostnames that are vague or temporary, such as:

- `server`
- `ubuntu`
- `test`
- `new-vm`
- `main`
- `final`
- `proxmox-main`

## Naming Patterns

### Router and Firewall

Use:

- `edge`

FQDN:

- `edge.home.lab`

Reason:

- `edge` identifies the router/firewall at the network edge.
- This name remains stable even if the hardware changes later.

### Proxmox Hosts

Use:

- `pve01`
- `pve02`
- `pve03`

FQDN examples:

- `pve01.home.lab`
- `pve02.home.lab`
- `pve03.home.lab`

Reason:

- `pve` clearly identifies Proxmox VE hosts.
- Numbering keeps the names sortable and expandable.

### Network Devices

Use short device or role names.

Current examples:

- `tl-sg108e`

Future examples:

- `sw01`
- `sw02`
- `ap01`
- `ap02`

Reason:

- Switches and access points should be easy to identify during network troubleshooting.
- Numbering allows additional devices later.

### Admin Devices

Use simple admin-device names.

Current examples:

- `ws01`
- `laptop01`

FQDN examples:

- `ws01.home.lab`
- `laptop01.home.lab`

Reason:

- `ws01` identifies the main admin workstation.
- `laptop01` identifies the admin laptop without tying the hostname to a specific hardware model.

### Infrastructure Service VMs

Use role plus two-digit number:

- `<role>01`

Current and planned examples:

- `dns01`
- `dhcp01`
- `ntp01`
- `monitor01`
- `log01`
- `backup01`
- `zammad01`

FQDN examples:

- `dns01.home.lab`
- `backup01.home.lab`
- `monitor01.home.lab`

Reason:

- Role-based names make the service purpose obvious.
- Numbering allows future secondary systems such as `dns02` or `backup02`.

### Client and Test Devices

Use:

- `client01`
- `client02`
- `test01`
- `test02`

Reason:

- Client and test systems should not be confused with infrastructure systems.
- These names are useful for validation and troubleshooting scenarios.

### DMZ Systems

Use service role names when DMZ systems are introduced later.

Future examples:

- `proxy01`
- `web01`
- `app01`

Reason:

- DMZ systems should clearly show their purpose.
- Public-facing or semi-isolated systems should not be confused with internal server systems.

## Current Approved Hostnames

- `edge` — `edge.home.lab` — VyOS router/firewall — Approved
- `tl-sg108e` — `tl-sg108e.home.lab` — TP-Link managed switch — Approved
- `pve01` — `pve01.home.lab` — Proxmox host 1 — Approved
- `pve02` — `pve02.home.lab` — Proxmox host 2 — Approved
- `pve03` — `pve03.home.lab` — Nested Proxmox host — Approved
- `ws01` — `ws01.home.lab` — Admin workstation — Approved
- `laptop01` — `laptop01.home.lab` — Admin laptop — Approved
- `dns01` — `dns01.home.lab` — Internal DNS service — Approved
- `dhcp01` — `dhcp01.home.lab` — Future DHCP service if moved from VyOS — Planned
- `ntp01` — `ntp01.home.lab` — Future NTP service — Planned
- `monitor01` — `monitor01.home.lab` — Future monitoring service — Planned
- `log01` — `log01.home.lab` — Future central logging service — Planned
- `backup01` — `backup01.home.lab` — Future backup service — Planned
- `zammad01` — `zammad01.home.lab` — Future Zammad service desk — Planned

## Change Rule

Do not rename hosts casually after DNS, monitoring, backup jobs, or documentation refer to them.

If a hostname must change, record:

- old hostname
- new hostname
- reason for rename
- affected DNS records
- affected monitoring checks
- affected documentation
- validation result after the change

