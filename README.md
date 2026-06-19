# Homelab — Linux Infrastructure Operations Portfolio

A production-style Linux homelab for practicing and documenting real system administration, cloud support, networking, and infrastructure operations work.

This repository is my main hands-on portfolio for Linux System Administrator, Cloud Support Engineer, Infrastructure Support, Junior DevOps, and NOC/Operations roles.

---

## Purpose

The goal is to build a realistic infrastructure environment and document it like an operations team would: inventory, topology, services, validation tests, troubleshooting notes, change records, incidents, backups, monitoring, logging, and automation.

This project focuses on proving that services are not only installed, but also documented, tested, recoverable, and supportable.

---

## Lab Focus Areas

- Linux system administration
- Networking: IP addressing, routing, DNS, DHCP, DDNS, firewalling, VLAN planning
- Proxmox virtualization
- VyOS router/firewall practice
- Service documentation and runbooks
- Backup and restore validation
- Monitoring, logging, and alerting
- Incident response and change management
- Zammad ticketing workflow
- Bash/Python automation foundations
- Ansible, Docker Compose, CI/CD validation, and later Terraform/OpenTofu and k3s
- GitHub-based infrastructure documentation

---

## Current Environment

- Dell PowerEdge R610 / Proxmox host
- VyOS router/firewall
- TP-Link managed switch
- Ubuntu admin workstation
- Linux virtual machines for infrastructure services
- Current lab subnet: `10.10.0.0/24`

---

## Repository Structure

```text
homelab/
├── README.md
├── REPO_STRUCTURE.md
├── docs/
│   ├── 00-overview/
│   │   ├── project-roadmap.md
│   │   ├── dependency-order.md
│   │   └── recruiter-summary.md
│   │
│   ├── 01-inventory/
│   │   ├── asset-inventory.md
│   │   ├── vm-inventory.md
│   │   ├── service-inventory.md
│   │   └── unknowns.md
│   │
│   ├── 02-network/
│   │   ├── topology.md
│   │   ├── physical-topology.md
│   │   ├── ip-plan.md
│   │   ├── vlan-plan.md
│   │   ├── dns-plan.md
│   │   └── firewall-policy.md
│   │
│   ├── 03-services/
│   │   ├── dns.md
│   │   ├── dhcp.md
│   │   ├── ntp.md
│   │   ├── dhcp-ddns.md
│   │   ├── reverse-proxy-tls.md
│   │   ├── mail.md
│   │   ├── identity-access.md
│   │   ├── storage.md
│   │   ├── backup-restore.md
│   │   ├── monitoring.md
│   │   ├── logging.md
│   │   └── zammad.md
│   │
│   ├── 04-operations/
│   │   ├── validation.md
│   │   ├── change-log.md
│   │   ├── incidents.md
│   │   ├── security-baseline.md
│   │   └── service-catalog.md
│   │
│   ├── phases/
│   │   ├── phase-00-physical-lab-foundation.md
│   │   ├── phase-01-basic-network-connectivity.md
│   │   ├── phase-02-discovery-inventory.md
│   │   ├── phase-03-proxmox-virtualization-platform.md
│   │   ├── phase-04-router-firewall-foundation.md
│   │   ├── phase-05-ip-plan-naming-network-zones.md
│   │   ├── phase-06-dns-service.md
│   │   ├── phase-07-dhcp-service.md
│   │   ├── phase-08-ntp-time-service.md
│   │   ├── phase-09-dhcp-ddns-external-ddns.md
│   │   ├── phase-10-reverse-proxy-tls.md
│   │   ├── phase-11-mail-service.md
│   │   ├── phase-12-identity-access-control.md
│   │   ├── phase-13-file-storage-service.md
│   │   ├── phase-14-backup-restore.md
│   │   ├── phase-15-monitoring-alerting.md
│   │   ├── phase-16-central-logging.md
│   │   ├── phase-17-security-baseline.md
│   │   ├── phase-18-change-management.md
│   │   ├── phase-19-zammad-ticketing.md
│   │   ├── phase-20-incident-response.md
│   │   ├── phase-21-os-installation-automation.md
│   │   ├── phase-22-infrastructure-service-automation.md
│   │   ├── phase-23-software-application-deployment-pipeline.md
│   │   ├── phase-24-terraform-opentofu-iac.md
│   │   ├── phase-25-kubernetes-k3s-learning-cluster.md
│   │   └── phase-26-portfolio-polish.md
│   │
│   └── runbooks/
│       ├── daily-health-check.md
│       ├── network-troubleshooting.md
│       ├── dns-troubleshooting.md
│       ├── backup-restore-test.md
│       ├── incident-response.md
│       └── service-deployment-checklist.md
│
├── diagrams/
│   ├── current-topology.drawio
│   ├── target-topology.drawio
│   └── vlan-design.drawio
│
├── configs/
│   ├── vyos/
│   ├── dns/
│   ├── dhcp/
│   ├── ntp/
│   ├── reverse-proxy/
│   ├── mail/
│   ├── monitoring/
│   └── logging/
│
├── scripts/
│   ├── bash/
│   └── python/
│
├── automation/
│   ├── ansible/
│   ├── docker-compose/
│   ├── terraform-opentofu/
│   └── ci-cd/
│
├── evidence/
│   ├── phase-00-physical-lab-foundation/
│   ├── phase-01-basic-network-connectivity/
│   ├── phase-02-discovery-inventory/
│   ├── phase-03-proxmox-virtualization-platform/
│   ├── phase-04-router-firewall-foundation/
│   ├── dns-dhcp-ntp/
│   ├── backup-restore/
│   ├── monitoring-logging/
│   ├── security/
│   ├── incidents/
│   └── portfolio-screenshots/
│
└── templates/
    ├── phase-template.md
    ├── validation-evidence-template.md
    ├── change-record-template.md
    ├── incident-report-template.md
    └── runbook-template.md
```


---

## Documentation

- [Inventory](docs/inventory.md)
- [Network Topology](docs/topology.md)
- [IP Address Plan](docs/ip-plan.md)
- [Naming Conventions](docs/naming-conventions.md)
- [Service Catalog](docs/service-catalog.md)
- [Validation Tests](docs/validation.md)
- [Change Log](docs/change-log.md)
- [Incident Notes](docs/incidents.md)
- [Access Control](docs/access-control.md)
- [Backup Strategy](docs/backup-strategy.md)
- [Monitoring Strategy](docs/monitoring-strategy.md)
- [Logging Strategy](docs/logging-strategy.md)
- [Portfolio Evidence](docs/portfolio-evidence.md)
- [Runbooks](docs/runbooks/)
- [Project Phases](docs/phases/)

---

## Project Roadmap

This roadmap follows the detailed homelab phase summary and uses GitHub Issues to track one issue per phase.

| Phase | Focus |
|---|---|
| 0 | Physical lab foundation: place, power, cable, and label equipment |
| 1 | Basic network connectivity: router, switch, R610, and admin machines |
| 2 | Discovery inventory: hardware, IPs, OS, VMs, services, and unknowns |
| 3 | Proxmox virtualization platform on the R610 |
| 4 | VyOS router/firewall foundation |
| 5 | IP plan, naming, subnets, VLANs, and network zones |
| 6 | Internal DNS service |
| 7 | DHCP service with scopes and reservations |
| 8 | NTP/time service for reliable logs and systems |
| 9 | DHCP-DDNS and external DDNS |
| 10 | Reverse proxy and TLS |
| 11 | Mail service, starting carefully with internal mail |
| 12 | Identity and access control: users, groups, SSH, sudo, service accounts |
| 13 | File/storage service with NFS, Samba, TrueNAS, or Linux file server |
| 14 | Backup and restore validation |
| 15 | Monitoring and alerting |
| 16 | Central logging |
| 17 | Security baseline: SSH, firewall, hardening, exposure review |
| 18 | Change management: reason, risk, rollback, validation, result |
| 19 | Zammad ticketing for requests, incidents, changes, and service problems |
| 20 | Incident response practice and documentation |
| 21 | OS installation automation with Proxmox templates, cloud-init, PXE/autoinstall |
| 22 | Infrastructure service automation with Ansible, Docker Compose, and CI/CD validation |
| 23 | Software/application deployment pipeline |
| 24 | Terraform/OpenTofu infrastructure as code |
| 25 | Kubernetes/k3s learning cluster |
| 26 | Portfolio polish for recruiter readability and evidence |

---

## Validation Approach

Every important change should include validation.

Example checks:

```bash
ip addr
ip route
systemctl status <service>
journalctl -u <service> --since "10 minutes ago"
ss -tulpn
dig <hostname>
curl -I <service-url>
```

A task is considered complete only when it has:

- documentation
- validation command/output
- evidence when useful
- change note or commit history
- rollback or troubleshooting note when needed

---

## Evidence

Evidence is stored in `evidence/` and may include:

- command output from validation tests
- screenshots of dashboards or service status
- backup and restore test logs
- DNS/DHCP test results
- network connectivity tests
- incident troubleshooting notes
- sanitized configuration examples

Example names:

```text
2026-06-19-host-inventory.txt
2026-06-19-network-validation.txt
2026-06-19-dns-test.txt
2026-06-19-backup-restore-test.txt
```

---

## Skills Demonstrated

- Linux administration
- TCP/IP networking
- DNS, DHCP, DDNS, routing, and firewall basics
- Proxmox virtualization
- VyOS router/firewall operations
- Service deployment and troubleshooting
- Backup and restore validation
- Monitoring, logging, and alerting foundations
- Incident, change, and ticketing workflow
- Technical documentation
- Bash/Python scripting foundations
- Git and GitHub workflow
- Infrastructure automation foundations

---

## Status

This project is actively being built and documented.

Current priority:

1. Complete physical lab, inventory, and topology documentation
2. Add validation evidence from the live lab
3. Build core network services: DNS, DHCP, NTP, and DDNS
4. Add monitoring, logging, backup, incident, and change documentation
5. Keep the repository recruiter-readable and evidence-based

---

## About Me

I am an LFCS-certified Linux administrator based in Korea, preparing for Linux System Administrator, Cloud Support Engineer, Infrastructure Operations, and Junior DevOps roles.

GitHub: [MichelZOGBELEMOU](https://github.com/MichelZOGBELEMOU)
