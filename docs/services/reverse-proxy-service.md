# Reverse Proxy Service

## Service Summary

Service name:

- HAProxy reverse proxy

Current node:

- `haproxy01`

Planned second node:

- `haproxy02`

Systemd service:

- `haproxy.service`

Main configuration file:

- `/etc/haproxy/haproxy.cfg`

TLS certificate file:

- `/etc/haproxy/certs/files.home.lab.pem`

Published service:

- File Browser

Published URL:

- `https://files.home.lab`

Backend:

- `fb01`
- `10.10.20.100:8080`

## Purpose

The HAProxy reverse proxy provides a controlled HTTPS entry point for selected internal web services.

For Phase 10, it publishes File Browser through `files.home.lab`.

The service demonstrates:

- Reverse proxy frontend/backend design.
- HTTP-to-HTTPS redirection.
- TLS termination.
- Host-based routing.
- Backend service forwarding.
- Basic health checking.
- Operational validation through systemd and logs.

## Current Service Status Evidence

Observed status:

    haproxy.service - HAProxy Load Balancer
    Loaded: loaded (/usr/lib/systemd/system/haproxy.service; enabled; preset: enabled)
    Active: active (running) since Thu 2026-09-10 21:51:10 KST
    Status: "Ready."

Runtime process:

    /usr/sbin/haproxy -Ws -f /etc/haproxy/haproxy.cfg -p /run/haproxy.pid -S /run/haproxy-master.sock

Observed result:

- HAProxy is installed.
- HAProxy is enabled at boot.
- HAProxy is active and running.
- HAProxy has loaded `/etc/haproxy/haproxy.cfg`.
- HAProxy is processing HTTPS requests.

## Current HAProxy Frontends

### HTTP Redirect Frontend

Name:

- `filebrowserhttp`

Purpose:

- Listen on HTTP port 80.
- Redirect clients to HTTPS.

Configuration:

    frontend filebrowserhttp
      bind :80
      mode http
      http-request redirect scheme https code 301

Expected behavior:

- `http://files.home.lab` redirects to `https://files.home.lab`.

### HTTPS Frontend

Name:

- `https_front`

Purpose:

- Listen on HTTPS port 443.
- Terminate TLS.
- Route requests based on Host header.

Configuration:

    frontend https_front
      bind *:443 ssl crt /etc/haproxy/certs/files.home.lab.pem
      mode http

      acl host_filebrowser hdr(host) -i files.home.lab
      use_backend filebrowser_backend if host_filebrowser

Expected behavior:

- Requests with Host header `files.home.lab` are sent to `filebrowser_backend`.

## Current HAProxy Backend

Backend name:

- `filebrowser_backend`

Backend server:

- `fb01`

Backend address:

- `10.10.20.100:8080`

Configuration:

    backend filebrowser_backend
      mode http
      option httpchk GET /
      http-request set-header X-Forwarded-Proto https
      http-request set-header X-Forwarded-For %[src]
      server fb01 10.10.20.100:8080

Expected behavior:

- HAProxy forwards requests to File Browser.
- HAProxy performs an HTTP health check with `GET /`.
## File Browser Backend Service

Application:

- File Browser

Version:

- `File Browser v2.63.23/833d9088`

Backend IP:

- `10.10.20.100`

Backend port:

- `8080`

Systemd unit:

    [Unit]
    Description=File Browser
    Documentation=https://github.com/filebrowser/filebrowser
    Wants=network-online.target
    After=network-online.target

    [Service]
    Type=simple
    User=filebrowser
    Group=filebrowser
    WorkingDirectory=/var/filebrowser

    ExecStart=/usr/bin/filebrowser --address 10.10.20.100 --port 8080  --config /etc/filebrowser/filebrowser.yml -d /var/lib/filebrowser/filebrowser.db

    Restart=on-failure
    RestartSec=5s

    NoNewPrivileges=true

    [Install]
    WantedBy=multi-user.target

Service design notes:

- File Browser runs as the `filebrowser` user.
- File Browser runs as the `filebrowser` group.
- Working directory is `/var/filebrowser`.
- Configuration file is `/etc/filebrowser/filebrowser.yml`.
- Database file is `/var/lib/filebrowser/filebrowser.db`.
- Service restarts automatically on failure.
- `NoNewPrivileges=true` is enabled.

## File Browser Binary Evidence

Observed binary check:

    ldd filebrowser

Observed result:

    linux-vdso.so.1
    libc.so.6 => /lib/x86_64-linux-gnu/libc.so.6
    /lib64/ld-linux-x86-64.so.2

Observed file type:

    filebrowser: ELF 64-bit LSB executable, x86-64, dynamically linked, interpreter /lib64/ld-linux-x86-64.so.2, stripped

Observed version:

    File Browser v2.63.23/833d9088

This confirms:

- The application is a Linux x86-64 executable.
- The application binary runs directly on the host.
- The deployed version is known and documented.

## Log Evidence

Observed HAProxy logs show successful requests:

    https_front~ filebrowser_backend/fb01 ... 200

Example client IP:

- `10.10.30.10`

Observed backend:

- `filebrowser_backend/fb01`

Observed HTTP status:

- `200`

Meaning:

- Client requests reached HAProxy.
- HAProxy selected the HTTPS frontend.
- HAProxy routed traffic to `filebrowser_backend`.
- Backend server `fb01` responded successfully.
- File Browser was reachable through the reverse proxy.

## Operational Commands

Check HAProxy service:

    systemctl status haproxy --no-pager

Validate HAProxy config:

    sudo haproxy -c -f /etc/haproxy/haproxy.cfg

Reload HAProxy safely:

    sudo systemctl reload haproxy

Restart HAProxy:

    sudo systemctl restart haproxy

View HAProxy logs:

    sudo journalctl -u haproxy -n 100 --no-pager

Follow HAProxy logs:

    sudo journalctl -u haproxy -f

Check listening ports:

    ss -tulpn | grep haproxy

Test HTTP redirect:

    curl -I http://files.home.lab

Test HTTPS:

    curl -I https://files.home.lab

Test with certificate verification disabled for troubleshooting only:

    curl -kI https://files.home.lab

## Backend Operational Commands

Check File Browser service:

    systemctl status filebrowser --no-pager

Restart File Browser:

    sudo systemctl restart filebrowser

View File Browser logs:

    sudo journalctl -u filebrowser -n 100 --no-pager

Test backend directly from HAProxy host:

    curl -I http://10.10.20.100:8080

## Backup Requirements

Important files to back up:

- `/etc/haproxy/haproxy.cfg`
- `/etc/haproxy/certs/files.home.lab.pem`
- `/etc/filebrowser/filebrowser.yml`
- `/var/lib/filebrowser/filebrowser.db`
- File Browser data directory, depending on configured storage path
- File Browser systemd unit file

Before changing HAProxy config:

    sudo cp /etc/haproxy/haproxy.cfg /etc/haproxy/haproxy.cfg.backup.$(date +%F-%H%M)

Before changing File Browser service:

    sudo systemctl cat filebrowser

## Future haproxy02 Notes

When `haproxy02` is added, document:

- Hostname
- IP address
- HAProxy version
- Certificate location
- Config sync method
- DNS/failover method
- Validation test showing service works through `haproxy02`
- Failover test from `haproxy01` to `haproxy02`

Possible future config consistency check:

    diff -u haproxy01-haproxy.cfg haproxy02-haproxy.cfg

Do not assume HA is complete just because a second HAProxy node exists. HA is only proven after failover validation.

## Known Issues / Follow-Ups

- Document the IP address of `haproxy01`.
- Document DNS record for `files.home.lab`.
- Document whether clients can access `10.10.20.100:8080` directly.
- Add `haproxy02` plan after single-node validation is complete.
- Add monitoring later for HAProxy service, TLS certificate expiry, and backend health.

## Change Log

- 2026-09-10: HAProxy active and routing `files.home.lab` to File Browser backend `10.10.20.100:8080`.
- Future: Add `haproxy02` for redundancy testing.

