# Reverse Proxy Validation

## Purpose

This document records validation evidence for Phase 10 reverse proxy and TLS work.

Current validation target:

- Reverse proxy: HAProxy
- Current node: `haproxy01`
- Planned second node: `haproxy02`
- Published service: File Browser
- Service URL: `https://files.home.lab`
- Backend: `10.10.20.100:8080`

## Validation Summary

Current observed result:

- HAProxy service is active and running.
- HAProxy is enabled at boot.
- HAProxy is listening with HTTPS frontend configuration.
- File Browser backend is reachable through HAProxy.
- HAProxy logs show successful `200` responses from `filebrowser_backend/fb01`.

## Test 1: HAProxy Service Status

Command:

    systemctl status haproxy --no-pager

Observed evidence:

    haproxy.service - HAProxy Load Balancer
    Loaded: loaded (/usr/lib/systemd/system/haproxy.service; enabled; preset: enabled)
    Active: active (running) since Thu 2026-09-10 21:51:10 KST
    Status: "Ready."

Result:

- Status: Pass
- HAProxy is active.
- HAProxy is enabled.
- HAProxy loaded `/etc/haproxy/haproxy.cfg`.

## Test 2: HAProxy Runtime Process

Observed process:

    /usr/sbin/haproxy -Ws -f /etc/haproxy/haproxy.cfg -p /run/haproxy.pid -S /run/haproxy-master.sock

Result:

- Status: Pass
- HAProxy is running with the expected config file.
- HAProxy master socket is enabled.

## Test 3: HTTP Frontend Redirect

Configured frontend:

    frontend filebrowserhttp
      bind :80
      mode http
      http-request redirect scheme https code 301

Validation command:

    curl -I http://files.home.lab

Expected result:

- HTTP status should show redirect.
- Expected redirect code: `301`
- Location should point to `https://files.home.lab`

Result:

- Status: To be recorded
- Observed status: TBD
- Observed Location header: TBD

## Test 4: HTTPS Frontend Configuration

Configured frontend:

    frontend https_front
      bind *:443 ssl crt /etc/haproxy/certs/files.home.lab.pem
      mode http

      acl host_filebrowser hdr(host) -i files.home.lab
      use_backend filebrowser_backend if host_filebrowser

Validation command:

    curl -I https://files.home.lab

Expected result:

- HTTPS connection succeeds.
- Request with Host header `files.home.lab` reaches the File Browser backend.
- Response is returned from File Browser.

Result:

- Status: Pass based on HAProxy logs showing `200` responses
- Additional curl output: To be recorded

## Test 5: Backend Routing

Configured backend:

    backend filebrowser_backend
      mode http
      option httpchk GET /
      http-request set-header X-Forwarded-Proto https
      http-request set-header X-Forwarde-For %[src]
      server fb01 10.10.20.100:8080

Observed log evidence:

    https_front~ filebrowser_backend/fb01 ... 200

Client IP observed:

- `10.10.30.10`

Backend observed:

- `filebrowser_backend/fb01`

HTTP status observed:

- `200`

Result:

- Status: Pass
- HAProxy is routing HTTPS frontend traffic to File Browser backend.
- File Browser is returning successful HTTP responses.

## Test 6: File Browser Service Definition

Observed systemd unit:

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

Validation points:

- Runs as non-root user: yes
- Uses systemd: yes
- Binds to expected backend IP: `10.10.20.100`
- Binds to expected backend port: `8080`
- Uses config file: `/etc/filebrowser/filebrowser.yml`
- Uses database: `/var/lib/filebrowser/filebrowser.db`
- Restarts on failure: yes
- Uses `NoNewPrivileges=true`: yes

Result:

- Status: Pass

## Test 7: File Browser Binary Verification

Observed command:

    file filebrowser

Observed result:

    filebrowser: ELF 64-bit LSB executable, x86-64, version 1 (SYSV), dynamically linked, interpreter /lib64/ld-linux-x86-64.so.2, stripped

Observed command:

    ./filebrowser version

Observed result:

    File Browser v2.63.23/833d9088

Result:

- Status: Pass
- Application binary is identified.
- Version is documented.

## Test 8: TLS Certificate Check

Command:

    echo | openssl s_client -connect files.home.lab:443 -servername files.home.lab 2>/dev/null | openssl x509 -noout -subject -issuer -dates

Expected result:

- Certificate is presented by HAProxy.
- Certificate is valid for `files.home.lab`.
- Issuer and expiry dates are documented.

Result:

- Status: To be recorded
- Subject: TBD
- Issuer: TBD
- Not before: TBD
- Not after: TBD

## Test 9: HAProxy Config Validation

Command:

    sudo haproxy -c -f /etc/haproxy/haproxy.cfg

Expected result:

- HAProxy reports configuration is valid.

Result:

- Status: To be recorded
- Output: TBD

Important follow-up:

The current backend contains this line:

    http-request set-header X-Forwarde-For %[src]

This should likely be corrected to:

    http-request set-header X-Forwarded-For %[src]

After correction, rerun:

    sudo haproxy -c -f /etc/haproxy/haproxy.cfg
    sudo systemctl reload haproxy

## Test 10: Backend Direct Reachability from HAProxy

Command to run from `haproxy01`:

    curl -I http://10.10.20.100:8080

Expected result:

- File Browser returns a valid HTTP response.

Result:

- Status: To be recorded
- Observed HTTP status: TBD

## Test 11: Direct Backend Access from Client Network

Purpose:

Confirm whether clients can bypass HAProxy and access File Browser directly.

Command from client:

    curl -I http://10.10.20.100:8080

Expected secure result:

- Direct backend access should be blocked or intentionally documented.

Result:

- Status: To be recorded
- If allowed, document why.
- If blocked, document firewall rule behavior.

## Test 12: Browser Test

Test URL:

    https://files.home.lab

Expected result:

- File Browser web UI loads.
- Login page or expected File Browser page is visible.
- Browser certificate behavior is understood.
- No unexpected backend error is shown.

Result:

- Status: To be recorded
- Client used: TBD
- Browser used: TBD
- Certificate warning: yes/no/TBD

## Test 13: Log Validation

Command:

    sudo journalctl -u haproxy -n 100 --no-pager

Observed evidence:

    10.10.30.10:50604 [10/Sep/2026:21:52:49.354] https_front~ filebrowser_backend/fb01 ... 200

Meaning:

- Client `10.10.30.10` reached HAProxy.
- HAProxy used `https_front`.
- HAProxy selected `filebrowser_backend`.
- Backend server `fb01` answered.
- Response status was `200`.

Result:

- Status: Pass

## Test 14: Planned haproxy02 Validation

This test is not complete yet because `haproxy02` is planned.

When `haproxy02` is added, validate:

- HAProxy installed on `haproxy02`.
- Same or intentionally equivalent HAProxy config exists.
- TLS certificate exists on `haproxy02`.
- `haproxy02` can reach File Browser backend.
- `https://files.home.lab` can work through `haproxy02`.
- Failover method is tested.

Future commands:

    systemctl status haproxy --no-pager
    sudo haproxy -c -f /etc/haproxy/haproxy.cfg
    curl -I https://files.home.lab
    sudo journalctl -u haproxy -n 100 --no-pager

Future result:

- Status: Planned

## Current Validation Status

Passed:

- HAProxy service active/running.
- HAProxy enabled.
- HAProxy processing requests.
- HTTPS frontend selects File Browser backend.
- File Browser backend returns HTTP 200.
- File Browser systemd service design is documented.
- File Browser binary and version are documented.

Needs recording:

- Exact DNS result for `files.home.lab`.
- HAProxy node IP address.
- TLS certificate subject, issuer, and expiry.
- HAProxy config validation output.
- HTTP redirect curl output.
- Direct backend access firewall behavior.
- Browser certificate trust behavior.

Needs correction:

- Change `X-Forwarde-For` to `X-Forwarded-For`.

Future:

- Add and validate `haproxy02`.
- Decide DNS/failover/virtual IP approach.
- Add monitoring for HAProxy and File Browser.
- Add backup/restore procedure for HAProxy config, certificates, and File Browser data.

## Acceptance Criteria

Phase 10 validation is complete when:

- [x] HAProxy is running.
- [x] HAProxy is enabled at boot.
- [x] HAProxy routes `files.home.lab` to File Browser.
- [x] File Browser returns successful responses through HAProxy.
- [x] File Browser service is documented.
- [x] File Browser version is documented.
- [ ] `files.home.lab` DNS result is documented.
- [ ] TLS certificate details are documented.
- [ ] HAProxy config validation command output is documented.
- [ ] HTTP-to-HTTPS redirect is tested and documented.
- [ ] Backend direct access behavior is documented.
- [ ] `haproxy02` plan is documented.
- [ ] Future failover validation is completed after `haproxy02` is deployed.

## Change Log

- 2026-09-10: HAProxy service active on `haproxy01`; File Browser routed through `https_front` to `filebrowser_backend/fb01`; backend returned HTTP 200.
- Future: Add `haproxy02` and validate redundancy/failover.

