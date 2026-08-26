# Time Sync Checks

## Purpose

This document records the operational checks used to verify time synchronization across the homelab.

Accurate time is required for logs, DNS troubleshooting, TLS certificates, authentication, monitoring, backups, and incident timelines.

## Scope

In scope:

- Proxmox hosts.
- Infrastructure VMs.
- Admin workstation `ws01`.
- Basic NTP/time sync validation.
- Evidence collection for future monitoring.

Out of scope:

- Building a dedicated internal NTP server.
- DHCP-delivered NTP options.
- Full monitoring alert implementation.

## Expected Standard

All infrastructure systems should have:

- Time synchronization enabled.
- System clock synchronized.
- Time zone documented.
- No major clock drift between systems.
- Failed synchronization recorded before remediation.

## Systems to Check

- `pve01` — Proxmox host.
- `pve02` — Proxmox host.
- `pve03` — Proxmox host.
- `dns01` — authoritative DNS.
- `dns02` — secondary authoritative DNS.
- `dns-resolv-01` — client resolver.
- `dns-resolv-02` — client resolver.
- `ws01` — admin workstation.

## Linux Time Sync Commands

Run on each Linux host or VM:

    timedatectl

Expected indicators:

    System clock synchronized: yes
    NTP service: active

If the host uses `systemd-timesyncd`, also check:

    systemctl status systemd-timesyncd --no-pager

Optional detailed check:

    timedatectl timesync-status

## Proxmox Host Checks

Run on each Proxmox host:

    timedatectl
    systemctl status chrony --no-pager || systemctl status systemd-timesyncd --no-pager

If Chrony is installed:

    chronyc sources -v
    chronyc tracking

Expected result:

- At least one valid time source is reachable.
- The selected source is marked with `*`.
- Offset and drift are not extreme.

## Cross-System Time Comparison

From `ws01`, compare time across key systems:

    ssh pve01.mgmt.home.lab date -Is
    ssh pve02.mgmt.home.lab date -Is
    ssh pve03.mgmt.home.lab date -Is
    ssh dns01.servers.home.lab date -Is
    ssh dns02.servers.home.lab date -Is
    ssh dns-resolv-01.servers.home.lab date -Is
    ssh dns-resolv-02.servers.home.lab date -Is
    date -Is

Expected result:

- All systems report the same date.
- Times are close enough for log correlation.
- Large differences are investigated before continuing service work.

## Evidence Template

Validation date:
Validation operator: Michel
Validation client: ws01

System:
Command: timedatectl
System clock synchronized:
NTP service:
Time zone:
Status: Passed / Failed
Notes:

Repeat this template for:

- `pve01`
- `pve02`
- `pve03`
- `dns01`
- `dns02`
- `dns-resolv-01`
- `dns-resolv-02`
- `ws01`

## Troubleshooting Guide

If `System clock synchronized` is `no`:

1. Confirm network connectivity.
2. Confirm DNS resolution works.
3. Check the time sync service state.
4. Check configured NTP sources.
5. Record the failed state before restarting services.

Useful commands:

    ping -c 4 1.1.1.1
    resolvectl query pool.ntp.org
    systemctl status systemd-timesyncd --no-pager
    systemctl status chrony --no-pager
    journalctl -u systemd-timesyncd --since "1 hour ago" --no-pager
    journalctl -u chrony --since "1 hour ago" --no-pager

If DNS works but NTP fails:

- Check firewall rules for outbound UDP/123.
- Check whether configured NTP servers are reachable.
- Check whether the host clock is too far from the correct time.

## Operational Use

Run these checks:

- After building a new VM.
- After changing DNS or firewall rules.
- Before troubleshooting distributed service issues.
- Before collecting incident evidence.
- Before enabling monitoring and logging services.

## Acceptance Criteria

- [ ] Each Proxmox host has time synchronization enabled.
- [ ] Each DNS system has time synchronization enabled.
- [ ] `ws01` has time synchronization enabled.
- [ ] Time status was checked using documented commands.
- [ ] Any failures were recorded with troubleshooting notes.
- [ ] Evidence is available for future monitoring and incident documentation.

## Final Result

Time synchronization checks provide a basic operations control for the homelab.

This document proves that accurate time is treated as shared infrastructure, not as an assumed default.

