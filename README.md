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

- HPE PROLIANT DL360 Gen10 / Proxmox host
- Samsung Desktop / second Proxmox host
- VyOS router/firewall
- TP-Link managed switch
- ipTIME AX2004T router (AP)
- Ubuntu admin workstation
- Linux virtual machines for infrastructure services
- Current network design: VLAN-based segmented network with separate MGMT, SRV, ADMIN, CLIENT, GST-IOT, and DMZ zones.


---

## Current Phase

The homelab is currently in Phase 5: VLAN and network-zone implementation.

Current focus:

- VLAN gateway configuration
- Network-zone separation
- Admin to Management access
- Firewall baseline
- Service-specific firewall rules added later as machines and services are installed

---


## Repository Structure

The repository is organized by operational artifact area. Some files already exist; others are planned and created phase by phase as evidence is produced.

```text
homelab/
├── README.md
├── LICENSE
├── CHANGELOG.md

├── docs/
│   ├── physical/
│   │   ├── physical-layout.md
│   │   └── cable-map.md
│   │
│   ├── inventory/
│   │   ├── homelab-inventory.md
│   │   ├── ip-inventory.md
│   │   └── known-unknowns.md
│   │
|   ├── network/
|   |   ├── router-firewall-foundation.md
|   |   ├── vlan-plan.md
|   |   ├── switch-port-map.md
|   |   ├── network-change-roolback.md
|   |   ├── current-firewall-policy.md
│   │   ├── initial-network-setup.md
│   │   ├── current-topology.md
│   │   ├── router-firewall-summary.md
│   │   ├── current-routing.md
│   │   ├── current-firewall-policy.md
│   │   ├── ip-plan.md
│   │   ├── hostname-standard.md
│   │   ├── network-zones.md
│   │   ├── dns-plan.md
│   │   ├── dns-records.md
│   │   ├── dhcp-plan.md
│   │   ├── dhcp-reservations.md
│   │   ├── dhcp-ddns-plan.md
│   │   ├── external-ddns-plan.md
│   │   ├── ddns-validation.md
│   │   ├── reverse-proxy-plan.md
│   │   ├── reverse-proxy-validation.md
│   │   └── mail-dns-records.md
│   │
│   ├── proxmox/
│   │   ├── proxmox-host-summary.md
│   │   ├── vm-inventory.md
│   │   └── ubuntu-template.md
│   │
│   ├── servers/
│   │   ├── r610-server-profile.md
│   │   └── hardware-risk-notes.md
│   │
│   ├── services/
│   │   ├── dns-service.md
│   │   ├── dhcp-service.md
│   │   ├── time-service.md
│   │   ├── dhcp-ddns-service.md
│   │   ├── reverse-proxy-service.md
│   │   ├── mail-service.md
│   │   ├── mail-requirements.md
│   │   └── zammad-service.md
│   │
│   ├── operations/
│   │   └── time-sync-checks.md
│   │
│   ├── security/
│   │   ├── identity-access-plan.md
│   │   ├── ssh-access-policy.md
│   │   ├── sudo-policy.md
│   │   ├── service-accounts.md
│   │   ├── tls-certificate-plan.md
│   │   ├── mail-security-notes.md
│   │   ├── security-baseline.md
│   │   ├── ssh-policy.md
│   │   ├── firewall-review.md
│   │   └── exposure-review.md
│   │
│   ├── storage/
│   │   ├── storage-service.md
│   │   ├── storage-layout.md
│   │   ├── access-permissions.md
│   │   └── storage-risk-notes.md
│   │
│   ├── backup/
│   │   ├── backup-policy.md
│   │   ├── backup-inventory.md
│   │   ├── restore-test-report.md
│   │   └── recovery-checklist.md
│   │
│   ├── monitoring/
│   │   ├── monitoring-plan.md
│   │   ├── service-checks.md
│   │   ├── alert-response.md
│   │   └── dashboard-notes.md
│   │
│   ├── logging/
│   │   ├── logging-plan.md
│   │   ├── log-sources.md
│   │   ├── log-retention.md
│   │   └── log-validation.md
│   │
│   ├── change-management/
│   │   ├── change-request-template.md
│   │   ├── rollback-plan-template.md
│   │   └── maintenance-window-template.md
│   │
│   ├── zammad/
│   │   ├── ticket-workflow.md
│   │   ├── ticket-categories.md
│   │   └── zammad-validation.md
│   │
│   ├── incidents/
│   │   ├── incident-template.md
│   │   ├── severity-levels.md
│   │   ├── incident-workflow.md
│   │   └── test-incident-report.md
│   │
│   ├── provisioning/
│   │   ├── manual-os-install.md
│   │   ├── cloud-init-plan.md
│   │   └── os-installation-automation.md
│   │
│   ├── automation/
│   │   ├── infrastructure-service-automation-plan.md
│   │   ├── ansible-plan.md
│   │   ├── docker-compose-plan.md
│   │   └── cicd-validation-plan.md
│   │
│   ├── deployment/
│   │   ├── application-deployment-plan.md
│   │   ├── cicd-pipeline.md
│   │   ├── rollback-procedure.md
│   │   └── deployment-validation.md
│   │
│   ├── iac/
│   │   ├── terraform-opentofu-plan.md
│   │   ├── proxmox-provider-notes.md
│   │   └── iac-validation.md
│   │
│   ├── kubernetes/
│   │   ├── k3s-cluster-plan.md
│   │   ├── k3s-installation.md
│   │   ├── k3s-validation.md
│   │   └── test-workload.md
│   │
│   ├── architecture.md (1/3)
|   ├── portfolio-summary.md
│   └── interview-stories.md
├── ansible/
│   ├── inventory.ini
│   └── playbooks/
├── compose/
│   └── <service>/
│       └── docker-compose.yml
├── terraform/
│   └── README.md
├── opentofu/
│   └── README.md
└── .github/
    └── workflows/
        ├── validation.yml
        ├── app-ci.yml
        └── app-deploy.yml

```


---




## Documentation

Current and planned documentation follows the same artifact paths used in the GitHub phase issues.

### Current committed evidence

- [Physical layout](docs/physical/physical-layout.md)
- [Cable map](docs/physical/cable-map.md)
- [Current network topology](docs/network/current-topology.md)
- [Initial network setup](docs/network/initial-network-setup.md)

### Phase 2: Discovery inventory

- `docs/inventory/homelab-inventory.md`
- `docs/inventory/ip-inventory.md`
- `docs/inventory/known-unknowns.md`

### Phase 3: Proxmox virtualization platform

- `docs/proxmox/proxmox-host-summary.md`
- `docs/proxmox/vm-inventory.md`
- `docs/servers/r610-server-profile.md`
- `docs/servers/hardware-risk-notes.md`

### Phase 4: Router/firewall foundation

- `docs/network/router-firewall-summary.md`
- `docs/network/current-routing.md`
- `docs/network/current-firewall-policy.md`

### Phase 5: IP plan, naming, and network zones

- `docs/network/ip-plan.md`
- `docs/network/hostname-standard.md`
- `docs/network/network-zones.md`

### Phase 6: DNS service

- `docs/network/dns-plan.md`
- `docs/services/dns-service.md`
- `docs/network/dns-records.md`

### Phase 7: DHCP service

- `docs/network/dhcp-plan.md`
- `docs/services/dhcp-service.md`
- `docs/network/dhcp-reservations.md`

### Phase 8: NTP/time service

- `docs/services/time-service.md`
- `docs/operations/time-sync-checks.md`

### Phase 9: DHCP-DDNS and external DDNS

- `docs/network/dhcp-ddns-plan.md`
- `docs/services/dhcp-ddns-service.md`
- `docs/network/external-ddns-plan.md`
- `docs/network/ddns-validation.md`

### Phase 10: Reverse proxy and TLS

- `docs/network/reverse-proxy-plan.md`
- `docs/services/reverse-proxy-service.md`
- `docs/security/tls-certificate-plan.md`
- `docs/network/reverse-proxy-validation.md`

### Phase 11: Mail service

- `docs/services/mail-service.md`
- `docs/services/mail-requirements.md`
- `docs/network/mail-dns-records.md`
- `docs/security/mail-security-notes.md`

### Phase 12: Identity and access control

- `docs/security/identity-access-plan.md`
- `docs/security/ssh-access-policy.md`
- `docs/security/sudo-policy.md`
- `docs/security/service-accounts.md`

### Phase 13: File/storage service

- `docs/storage/storage-service.md`
- `docs/storage/storage-layout.md`
- `docs/storage/access-permissions.md`
- `docs/storage/storage-risk-notes.md`

### Phase 14: Backup and restore

- `docs/backup/backup-policy.md`
- `docs/backup/backup-inventory.md`
- `docs/backup/restore-test-report.md`
- `docs/backup/recovery-checklist.md`

### Phase 15: Monitoring and alerting

- `docs/monitoring/monitoring-plan.md`
- `docs/monitoring/service-checks.md`
- `docs/monitoring/alert-response.md`
- `docs/monitoring/dashboard-notes.md`

### Phase 16: Central logging

- `docs/logging/logging-plan.md`
- `docs/logging/log-sources.md`
- `docs/logging/log-retention.md`
- `docs/logging/log-validation.md`

### Phase 17: Security baseline

- `docs/security/security-baseline.md`
- `docs/security/ssh-policy.md`
- `docs/security/firewall-review.md`
- `docs/security/exposure-review.md`

### Phase 18: Change management

- `docs/change-management/change-request-template.md`
- `docs/change-management/rollback-plan-template.md`
- `docs/change-management/maintenance-window-template.md`
- `CHANGELOG.md`

### Phase 19: Zammad ticketing

- `docs/services/zammad-service.md`
- `docs/zammad/ticket-workflow.md`
- `docs/zammad/ticket-categories.md`
- `docs/zammad/zammad-validation.md`

### Phase 20: Incident response

- `docs/incidents/incident-template.md`
- `docs/incidents/severity-levels.md`
- `docs/incidents/incident-workflow.md`
- `docs/incidents/test-incident-report.md`

### Phase 21: OS installation automation

- `docs/provisioning/manual-os-install.md`
- `docs/proxmox/ubuntu-template.md`
- `docs/provisioning/cloud-init-plan.md`
- `docs/provisioning/os-installation-automation.md`

### Phase 22: Infrastructure service automation

- `docs/automation/infrastructure-service-automation-plan.md`
- `docs/automation/ansible-plan.md`
- `docs/automation/docker-compose-plan.md`
- `docs/automation/cicd-validation-plan.md`

Code artifacts:

- `ansible/inventory.ini`
- `ansible/playbooks/base-linux.yml`
- `ansible/playbooks/<service>.yml`
- `compose/<service>/docker-compose.yml`
- `.github/workflows/validation.yml`

### Phase 23: Software/application deployment pipeline

- `docs/deployment/application-deployment-plan.md`
- `docs/deployment/cicd-pipeline.md`
- `docs/deployment/rollback-procedure.md`
- `docs/deployment/deployment-validation.md`

Code artifacts:

- `Dockerfile`
- `docker-compose.yml`
- `.github/workflows/app-ci.yml`
- `.github/workflows/app-deploy.yml`

### Phase 24: Terraform/OpenTofu infrastructure as code

- `terraform/` or `opentofu/`
- `docs/iac/terraform-opentofu-plan.md`
- `docs/iac/proxmox-provider-notes.md`
- `docs/iac/iac-validation.md`

### Phase 25: Kubernetes/k3s learning cluster

- `docs/kubernetes/k3s-cluster-plan.md`
- `docs/kubernetes/k3s-installation.md`
- `docs/kubernetes/k3s-validation.md`
- `docs/kubernetes/test-workload.md`

### Phase 26: Portfolio polish

- `README.md`
- `docs/architecture.md`
- `docs/portfolio-summary.md`
- `docs/interview-stories.md`

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
