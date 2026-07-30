# Home Lab

## Overview

This homelab provides the following core services:

- Network-wide DNS filtering (Pi-hole)
- Secure remote VPN access (WireGuard)
- Infrastructure monitoring (Uptime Kuma)

The environment runs on a single Intel NUC using Proxmox VE.

---

# Architecture

```text
                    Internet
                        │
        FiberHome HG6544C (ONT)
                        │
             ZTE ZXHN H3601P
                        │
                 192.168.1.1
                        │
                Proxmox Host
                 192.168.1.50
                        │
        ┌───────────────┼───────────────┐
        │               │               │
     CT100           CT101          CT102
    Pi-hole        WireGuard    Uptime Kuma
192.168.1.10    192.168.1.20   192.168.1.30
```

---

# Hardware

| Component | Value |
|----------|-------|
| Host | Intel NUC7i3DNB |
| CPU | Intel Core i3-7100U |
| RAM | 8 GB DDR4 |
| Storage | 128 GB NVMe SSD |
| Hypervisor | Proxmox VE 9.x |

---

# Network

| Item | Value |
|------|-------|
| LAN | 192.168.1.0/24 |
| Gateway | 192.168.1.1 |
| Proxmox | 192.168.1.50 |
| DNS | 192.168.1.10 |
| IPv6 | Disabled (DHCPv6 & RA) |

---

# Running Containers

| CT | Name | IP | Purpose |
|---:|------|----|---------|
|100|Pi-hole|192.168.1.10|DNS & Ad Blocking|
|101|WireGuard|192.168.1.20|Remote VPN|
|102|Uptime Kuma|192.168.1.30|Monitoring|

---

# Router

Model

```text
ZTE ZXHN H3601P
```

Port Forwarding

| Service | Protocol | Port |
|---------|----------|----:|
| WireGuard | UDP | 51820 |

DDNS

```text
vicvpn.go.ro
```

---

# Startup Automation

Startup orchestration

```text
/root/scripts/
```

Systemd service

```text
/etc/systemd/system/start-containers.service
```

Automatic restart

```cron
30 0 * * * /root/scripts/restart-proxmox.sh
```

---

# Backup

Weekly LXC backups.

Primary backup location

```text
Laptop
```

Restore location

```text
/var/lib/vz/dump/
```

---

# Documentation

| Document | Purpose |
|----------|---------|
| README.md | High-level overview |
| RECOVERY.md | Complete disaster recovery |
| SCRIPTS.md | Startup scripts, cron and systemd |
| NETWORK.md | Router and network configuration |
| BACKUP.md | Backup and restore procedures |

---

# Daily Health Check

```bash
pct list
systemctl status start-containers.service
systemctl is-enabled start-containers.service
crontab -l
tail -20 /var/log/start-containers.log

pct enter 101
wg show
```
