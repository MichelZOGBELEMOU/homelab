# NTP Service

## Service Summary

- Service name: Time synchronization / NTP.
- Service status: Planned / baseline client validation in progress.
- Service model: Hosts use local NTP clients such as Chrony or systemd-timesyncd.
- Dedicated internal NTP server: Not implemented yet.
- Network scope: All infrastructure zones.
- Phase: Phase 8 — NTP / time service.
- Owner: Michel.

## Purpose

The NTP service keeps infrastructure clocks accurate enough for operations work.

Accurate time supports:

- Log correlation across hosts.
- DNS and firewall troubleshooting.
- TLS certificate validation.
- Authentication and session auditing.
- Backup and restore evidence.
- Monitoring alerts and incident timelines.

## Current Design

The current design is intentionally simple.

Each host keeps its own clock synchronized using the operating system time synchronization service.

There is no dedicated internal NTP server yet.

This avoids adding another service dependency before the lab has monitoring, backups, and change-management practices in place.

## Service Boundary

This file defines the NTP service.

Operational commands and evidence templates are kept in:

- `docs/operations/time-sync-checks.md`

This avoids unnecessary duplication.

## In Scope

- Time synchronization as a required infrastructure control.
- Proxmox host time synchronization.
- Linux VM time synchronization.
- Admin workstation time synchronization.
- Basic expected behavior for clock accuracy.
- Future internal NTP planning.

## Out of Scope

- Building a dedicated internal NTP server now.
- DHCP-provided NTP options.
- NTP authentication.
- Monitoring alert rules.
- Automated remediation.

## Expected Behavior

All infrastructure systems should have:

- Time synchronization enabled.
- System clock synchronized.
- Time zone documented.
- No major clock drift between systems.
- Time status checked when logs are compared across hosts.

## Service Dependencies

Required dependencies:

- Working network connectivity.
- Working DNS resolution for external NTP sources.
- Firewall policy allowing required NTP traffic.
- Host-level time synchronization service installed and enabled.

Related documentation:

- `docs/operations/time-sync-checks.md`
- `docs/network/dns-validation.md`
- `docs/network/current-firewall-policiy.md`
- `docs/network/network-zones.md`
- `docs/proxmox/proxmox-host-summary.md`

## Expected Traffic

- UDP/123: NTP time synchronization.

Firewall expectation:

- Hosts using external NTP sources need outbound UDP/123.
- If internal NTP servers are added later, clients should use the internal servers instead of depending directly on external sources.

## Current Implementation Notes

Current state:

- Time synchronization is treated as a required baseline.
- Validation evidence belongs in `docs/operations/time-sync-checks.md`.
- No internal NTP server has been selected yet.

This is acceptable for the current stage because the immediate goal is to prove that important systems keep correct time before adding service complexity.

## Future Internal NTP Design

A later internal NTP design may use:

- `ntp01.servers.home.lab` as primary internal time source.
- `ntp02.servers.home.lab` as secondary internal time source.
- Chrony as the preferred implementation.
- DHCP-provided NTP server options.
- Monitoring checks for NTP reachability and clock offset.

Future model:

- Internal clients synchronize against `ntp01` and `ntp02`.
- Internal NTP servers synchronize against approved external sources.
- Monitoring alerts if time sync fails or drift becomes too large.

## Monitoring Notes

Future monitoring should check:

- Time synchronization service is active.
- Clock is synchronized.
- NTP source is reachable.
- Clock offset is within the chosen threshold.
- Important systems do not drift significantly from each other.

Monitoring implementation status:

- Deferred until monitoring phase.

## Backup and Recovery Notes

The NTP service has minimal data, but configuration still matters.

Future backup scope should include:

- Chrony configuration files, if used.
- systemd-timesyncd configuration files, if used.
- DHCP NTP option configuration, if later implemented.
- Monitoring rules for time synchronization, if later implemented.

Recovery expectation:

- A rebuilt host should automatically regain time synchronization from documented configuration.

Backup implementation status:

- Deferred until backup phase.

## Risks

If time synchronization fails:

- Logs from different systems may appear out of order.
- Incident timelines may be unreliable.
- TLS validation may fail.
- Authentication troubleshooting may become harder.
- Monitoring alerts may have misleading timestamps.
- Backup and restore evidence may be harder to trust.

## Implementation Checklist

- [ ] Confirm time synchronization method on each Proxmox host.
- [ ] Confirm time synchronization method on each infrastructure VM.
- [ ] Confirm time synchronization status on `ws01`.
- [ ] Record validation evidence in `docs/operations/time-sync-checks.md`.
- [ ] Confirm whether outbound UDP/123 is allowed where required.
- [ ] Decide later whether the lab needs dedicated internal NTP servers.

## Acceptance Criteria

- [ ] The lab has a documented time synchronization service definition.
- [ ] The lab has a separate operational checklist for time validation.
- [ ] Time synchronization status is validated on core infrastructure systems.
- [ ] Any time sync failures are documented before remediation.
- [ ] Future monitoring and backup expectations are documented.

## Final Result

The homelab treats time synchronization as a core infrastructure service.

At this stage, NTP is defined as a baseline host-level control, while detailed validation evidence is kept in `docs/operations/time-sync-checks.md`.

