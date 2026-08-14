# DHCP Service

## Service Summary

The `home.lab` network uses Kea DHCPv4 to provide DHCP service for selected VLANs.

DHCP is hosted on two Debian servers in VLAN 20 / Servers and reached from client VLANs through VyOS DHCP relay.

## Servers

| Host | IP address | Role | VLAN | Proxmox host |
|---|---:|---|---|---|
| `dhcp01.servers.home.lab` | `10.10.20.14` | Primary | VLAN 20 / Servers | `pve03` |
| `dhcp02.servers.home.lab` | `10.10.20.15` | Standby | VLAN 20 / Servers | `pve01` |

## Service Details

- DHCP software: Kea DHCPv4
- HA mode: hot-standby
- Systemd service: `isc-kea-dhcp4-server.service`
- Configuration file: `/etc/kea/kea-dhcp4.conf`
- Lease database: `/var/lib/kea/kea-leases4.csv`
- DHCP interface: `ens18`
- DHCP socket type: `udp`
- Kea HA/API port: TCP `8000`

## Network Placement

The DHCP servers are placed in VLAN 20 / Servers.

Client VLANs use DHCP relay because DHCP broadcast traffic does not cross VLAN boundaries by itself.

Current validated relay:

```text
service {
    dhcp-relay {
        listen-interface eth2.30
        server 10.10.20.14
        server 10.10.20.15
        upstream-interface eth2.20
    }
}
```

Current validated relay path:

- VLAN 30 / Admin to VLAN 20 / Servers
- Relay listen interface: `eth2.30`
- Relay upstream interface: `eth2.20`
- Relay targets:
  - `10.10.20.14`
  - `10.10.20.15`

## DHCP Scopes

| VLAN | Zone | Subnet | DHCP pool | Router | Status |
|---:|---|---|---|---|---|
| 10 | Management | `10.10.10.0/24` | `10.10.10.100 - 10.10.10.150` | `10.10.10.1` | Pending validation |
| 20 | Servers | `10.10.20.0/24` | None | `10.10.20.1` | Static only |
| 30 | Admin | `10.10.30.0/24` | `10.10.30.100 - 10.10.30.200` | `10.10.30.1` | Validated |
| 40 | Clients | `10.10.40.0/24` | `10.10.40.2 - 10.10.40.100` | `10.10.40.1` | Pending validation |
| 50 | Guest/IoT | `10.10.50.0/24` | `10.10.50.2 - 10.10.50.100` | `10.10.50.1` | Pending validation |
| 60 | DMZ | `10.10.60.0/24` | None | `10.10.60.1` | Deferred |

## DHCP Options

DHCP clients receive:

- DNS servers:
  - `10.10.20.12`
  - `10.10.20.13`
- Domain:
  - `home.lab`
- Router:
  - VLAN-specific gateway

## Reservations

Current reservations:

| Device | VLAN | MAC address | Reserved IP | Status |
|---|---:|---|---|---|
| `ws01` | 30 | `fe:cd:7a:10:bf:52` | `10.10.30.10` | Validated |
| Admin host | 30 | `BC:24:11:30:EB:75` | `10.10.30.11` | Pending verification |

Detailed reservation tracking:

- `docs/network/dhcp-reservations.md`

## High Availability

Kea DHCPv4 is configured in hot-standby mode.

Normal behavior:

- `dhcp01` acts as the primary DHCP server.
- `dhcp02` acts as the standby DHCP server.
- The servers exchange HA heartbeat messages.
- Client lease service should continue if the primary server is unavailable.

HA peer endpoints:

- `dhcp01`: `http://10.10.20.14:8000/`
- `dhcp02`: `http://10.10.20.15:8000/`

Required HA port:

- TCP `8000`

Server-specific HA identity:

- On `dhcp01`: `this-server-name` should be `dhcp01`.
- On `dhcp02`: `this-server-name` should be `dhcp02`.

## Validation Summary

Completed:

- [x] `dhcp01` hostname validated.
- [x] `dhcp01` FQDN validated.
- [x] `dhcp01` static network configuration validated.
- [x] Kea DHCPv4 service running.
- [x] Kea configuration syntax passed.
- [x] VLAN 30 DHCP relay validated.
- [x] VLAN 30 reservation for `ws01` validated.
- [x] DHCP HA failover tested for VLAN 30.
- [x] `dhcp02` allocated a VLAN 30 lease during controlled `dhcp01` outage.
- [x] HA heartbeat resumed after `dhcp01` was restored.

Pending:

- [ ] VLAN 10 DHCP validation.
- [ ] VLAN 40 DHCP validation.
- [ ] VLAN 50 DHCP validation.
- [ ] DNS option validation from a DHCP client.
- [ ] Domain option validation from a DHCP client.
- [ ] Firewall rule/counter evidence for DHCP relay.
- [ ] Firewall rule/counter evidence for Kea HA TCP `8000`.

Deferred:

- [ ] VLAN 60 DHCP scope.
- [ ] DHCP-DDNS integration.

## Operational Commands

Validate Kea config:


```bash
sudo kea-dhcp4 -t /etc/kea/kea-dhcp4.conf
```

Check DHCP service:

```bash
systemctl status isc-kea-dhcp4-server.service
```
Restart DHCP service:

```bash
sudo systemctl restart isc-kea-dhcp4-server.service
```

Check DHCP logs:

```bash
sudo journalctl -u isc-kea-dhcp4-server.service -n 50 --no-pager
```


Check lease database:

```bash
sudo tail -n 20 /var/lib/kea/kea-leases4.csv
```


Check Kea HA/API listener:

```bash
sudo ss -lntp | grep 8000
```

Check HA heartbeat logs:

```bash
sudo journalctl -u isc-kea-dhcp4-server.service --no-pager | grep -i "ha-heartbeat"
```
## Troubleshooting Notes

### Kea configuration syntax

A trailing comma in the Kea JSON configuration can prevent the service from validating or restarting.

Always validate before restarting:

```bash
sudo kea-dhcp4 -t /etc/kea/kea-dhcp4.conf 
```


### HA heartbeat warning during failover test

During the controlled outage of `dhcp01`, `dhcp02` logged heartbeat failures.

This was expected because the primary service was intentionally stopped.

Observed behavior:

- `dhcp02` detected loss of communication with `dhcp01`.
- `dhcp02` allocated a VLAN 30 lease.
- Heartbeat messages resumed after `dhcp01` was restored.

## Current Status

DHCP service is operational for the tested VLAN 30 scope.

Current result:

- VLAN 30 DHCP relay: PASS
- VLAN 30 reservation: PASS
- VLAN 30 HA failover: PASS

More validation is required before marking all DHCP-enabled VLANs complete.

