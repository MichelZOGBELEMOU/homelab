# File: docs/network/reverse-proxy-plan.md

# Reverse Proxy Plan

## Purpose

Phase 10 introduces a reverse proxy and TLS entry point for selected homelab web services.

For this phase, HAProxy was selected as the reverse proxy/load balancer. The first published service is a small GitHub-hosted application, File Browser, deployed internally and exposed through HAProxy using HTTPS.

This phase is also used as a learning bridge toward future CI/CD and deployment pipeline work. The goal is not only to make one service reachable, but to understand the manual deployment process first:

- Install and run an application as a Linux service.
- Place the application behind a reverse proxy.
- Add HTTPS/TLS.
- Validate logs, routing, backend reachability, and service behavior.
- Document the deployment path before automating it later.

## Scope

This phase covers:

- HAProxy reverse proxy deployment.
- HTTPS frontend configuration.
- HTTP-to-HTTPS redirection.
- File Browser backend publishing.
- TLS certificate usage for `files.home.lab`.
- Basic backend health checking.
- Reverse proxy validation.
- Documentation of the manual deployment process.

This phase does not yet cover:

- Full CI/CD pipeline automation.
- Docker deployment pipeline.
- Kubernetes ingress.
- Public internet exposure.
- Production-grade HAProxy clustering.
- Automated blue/green or rolling deployments.
- Full monitoring and alerting integration.

## Reverse Proxy Software Decision

Selected software:

- HAProxy

Reason:

- HAProxy is widely used in real infrastructure.
- It teaches clear frontend/backend separation.
- It supports HTTP routing, TLS termination, health checks, and load balancing.
- It is a strong foundation before learning deployment pipelines and high availability.
- It makes the request path easier to understand than hiding everything behind a higher-level automation platform too early.

## Current Reverse Proxy Node

Current node:

- Hostname: `haproxy01`
- Service: `haproxy.service`
- Config file: `/etc/haproxy/haproxy.cfg`
- Certificate path: `/etc/haproxy/certs/files.home.lab.pem`
- Current frontend: `https_front`
- Current backend: `filebrowser_backend`

Observed service status:

- Service: `haproxy.service`
- Status: active/running
- Enabled at boot: yes
- Main process: `/usr/sbin/haproxy`
- Runtime status: `Ready.`

## Planned Second Reverse Proxy Node

Planned node:

- Hostname: `haproxy02`
- Purpose: second HAProxy node for redundancy and future high availability testing

The planned second node should eventually allow testing:

- Configuration consistency between `haproxy01` and `haproxy02`
- Failover behavior
- Shared or duplicated TLS certificate handling
- DNS or virtual IP decision
- Load-balanced or active/passive reverse proxy design

Possible future HA design options:

- DNS with multiple A records:
  - `files.home.lab` points to both HAProxy nodes.
- Keepalived/VRRP:
  - One virtual IP floats between `haproxy01` and `haproxy02`.
- Manual failover:
  - DNS record moved from `haproxy01` to `haproxy02`.
- Router/firewall NAT change:
  - Only if public exposure is added later.

Initial recommendation:

Start with `haproxy01` working and fully documented. Add `haproxy02` only after the single-node route is stable and validated.

## Published Application

Application:

- File Browser

Reason for choosing it:

- Small project hosted on GitHub.
- Good practice for deploying a real application manually.
- Provides a simple web interface that can be proxied through HAProxy.
- Helps understand application deployment before building CI/CD pipelines later.

Current File Browser version:

- `File Browser v2.63.23/833d9088`

Application binary type:

- ELF 64-bit x86-64 executable
- Dynamically linked
- Runs as a Linux systemd service

## Current Service URL

User-facing service name:

- `files.home.lab`

Expected access method:

- `https://files.home.lab`

HTTP behavior:

- Port 80 redirects to HTTPS with HTTP 301.

HTTPS behavior:

- HAProxy listens on TCP/443.
- HAProxy terminates TLS using `/etc/haproxy/certs/files.home.lab.pem`.
- HAProxy forwards traffic to File Browser over HTTP.

## Current Traffic Flow

Request flow:

1. Client connects to `http://files.home.lab`.
2. HAProxy frontend `filebrowserhttp` listens on port 80.
3. HAProxy redirects the request to HTTPS.
4. Client connects to `https://files.home.lab`.
5. HAProxy frontend `https_front` listens on port 443.
6. HAProxy checks the HTTP Host header.
7. If host is `files.home.lab`, HAProxy sends the request to `filebrowser_backend`.
8. Backend `filebrowser_backend` forwards traffic to File Browser at `10.10.20.100:8080`.

Current backend:

- Backend name: `filebrowser_backend`
- Server name: `fb01`
- Backend IP: `10.10.20.100`
- Backend port: `8080`

## Current HAProxy Configuration Summary

Current HTTP frontend:

    frontend filebrowserhttp
      bind :80
      mode http
      http-request redirect scheme https code 301

Current HTTPS frontend:

    frontend https_front
      bind *:443 ssl crt /etc/haproxy/certs/files.home.lab.pem
      mode http

      acl host_filebrowser hdr(host) -i files.home.lab
      use_backend filebrowser_backend if host_filebrowser

Current backend:

    backend filebrowser_backend
      mode http
      option httpchk GET /
      http-request set-header X-Forwarded-Proto https
      http-request set-header X-Forwarded-For %[src]
      server fb01 10.10.20.100:8080

## DNS Plan

The service name should resolve to the HAProxy node, not directly to the File Browser backend.

Required DNS record:

- `files.home.lab` → HAProxy IP

Backend host/IP:

- File Browser backend: `10.10.20.100:8080`

Important design rule:

- Clients should access File Browser through `https://files.home.lab`.
- Clients should not need to remember or use `10.10.20.100:8080`.

When `haproxy02` is added, DNS design must be reviewed.

Possible future records:

- `haproxy01.servers.home.lab`
- `haproxy02.servers.home.lab`
- `files.home.lab`

## Firewall Plan

Minimum internal access paths:

- Client/Admin network to HAProxy:
  - TCP/80 for redirect
  - TCP/443 for HTTPS

- HAProxy to File Browser backend:
  - TCP/8080 to `10.10.20.100`

Recommended security direction:

- Allow users to access File Browser through HAProxy.
- Avoid exposing backend port `8080` directly to normal client networks.
- Keep public WAN access disabled unless a separate exposure review is completed.

## Manual Deployment Learning Goal

This phase intentionally uses a manual deployment process.

The learning goal is to understand:

- How a GitHub project becomes a running Linux service.
- How a binary is checked.
- How a systemd unit starts the application.
- How HAProxy routes requests to it.
- How TLS is attached at the reverse proxy layer.
- How logs prove that traffic is reaching the backend.
- What steps should later become automated in a pipeline.

This is important before building CI/CD because a pipeline automates a process that should already be understood manually.

## Future Pipeline Connection

This Phase 10 work prepares for a future deployment pipeline.

Manual steps that may later become automated:

- Download or build application artifact.
- Verify binary/version.
- Install binary to a standard path.
- Create or update systemd unit.
- Restart service safely.
- Validate local backend.
- Update HAProxy backend or route.
- Validate HTTPS endpoint.
- Roll back if validation fails.

Future pipeline should not be created until this manual process is documented and repeatable.

## Risks

Current risks:

- HAProxy is currently a single point of failure until `haproxy02` is added.
- File Browser backend availability depends on `10.10.20.100`.
- TLS certificate renewal/replacement process must be documented.
- Direct backend access may bypass HAProxy if firewall rules allow it.
- Public exposure should not be enabled without security review.

## Acceptance Criteria

Phase 10 is complete when:

- HAProxy is installed and running.
- HAProxy starts at boot.
- `files.home.lab` resolves to the HAProxy node.
- HTTP redirects to HTTPS.
- HTTPS works with the configured certificate.
- HAProxy forwards `files.home.lab` traffic to File Browser.
- File Browser runs as a systemd service.
- HAProxy logs show successful requests to `filebrowser_backend/fb01`.
- Backend health check is configured.
- Validation results are documented.
- The plan for `haproxy02` is documented as future redundancy work.

## Change Log

- 2026-09-10: HAProxy running on `haproxy01`; File Browser published through HTTPS frontend and `filebrowser_backend`.
- Future: Add `haproxy02` as second HAProxy node for redundancy.
