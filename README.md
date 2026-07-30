# Home Lab

# Static IP Allocation

| Device | IP |
|---------|----|
| Router | 192.168.1.1 |
| Proxmox | 192.168.1.50 |
| Pi-hole | 192.168.1.10 |
| WireGuard | 192.168.1.20 |
| Uptime Kuma | 192.168.1.30 |

## Hardware

- Intel NUC7i3DNB
- Intel Core i3-7100U
- 8 GB DDR4 RAM
- 128 GB NVMe SSD
- Proxmox VE

---

# Network

## Internet

```
Internet
    │
FiberHome HG6544C (ONT)
    │
ZTE ZXHN H3601P
    │
Proxmox
```

## LAN

```
Subnet: 192.168.1.0/24
Gateway: 192.168.1.1
```

## DNS

Primary DNS

```
192.168.1.10
```

## IPv6

```
DHCPv6: Disabled
RA: Disabled
```

---

# Proxmox

## Host

- Proxmox VE
- Local storage on NVMe SSD

---

# Containers

## CT100 - Pi-hole

**IP**

```
192.168.1.10
```

**Purpose**

- Local DNS
- Network-wide ad blocking

**Services**

- Pi-hole

---

## CT101 - WireGuard

**IP**

```
192.168.1.20
```

**Purpose**

- Remote VPN access

**Services**

- WireGuard

**Port**

```
UDP 51820
```

**DDNS**DIGI

```
vicvpn.go.ro
```

---

## CT102 - Uptime Kuma

**IP**

```
192.168.1.30
```

**Purpose**

- Service monitoring

**Monitors**

- Internet
- Pi-hole
- WireGuard

---

# Router

## Port Forwarding

| Service | Protocol | External Port | Internal Address | Internal Port |
|----------|----------|--------------:|------------------|--------------:|
| WireGuard | UDP | 51820 | 192.168.1.20 | 51820 |

---

# Backup

- Weekly Proxmox LXC backups
- Backup copies stored on laptop

---

# Proxmox Recovery

This procedure restores the Proxmox host after a clean installation.

---

# 1. Install Proxmox VE

Install the latest supported Proxmox VE version.

---

# 2. Configure the host

Hostname

```text
proxmox
```

Management IP

```text
192.168.1.50/24
```

Gateway

```text
192.168.1.1
```

DNS

```text
192.168.1.10
```

Timezone

```text
Europe/Bucharest
```

---

# 3. Restore LXC Containers

Restore the latest backups for:

```text
CT100 - Pi-hole
CT101 - WireGuard
CT102 - Uptime Kuma
CT103 - Expenses (when available)
```

Verify:

```bash
pct list
```

---

# 4. Restore Startup Scripts

Create the scripts directory.

```bash
mkdir -p /root/scripts
```

Copy the following files to:

```text
/root/scripts/
```

Files

```text
restart-proxmox.sh
shutdown-proxmox.sh
start-containers.sh
```

Make them executable.

```bash
chmod +x /root/scripts/*.sh
```

Verify.

```bash
ls -l /root/scripts
```

---

# 5. Restore systemd Service

Copy

```text
start-containers.service
```

to

```text
/etc/systemd/system/start-containers.service
```

Reload systemd.

```bash
systemctl daemon-reload
```

Enable the service.

```bash
systemctl enable start-containers.service
```

Verify.

```bash
systemctl is-enabled start-containers.service
```

Expected

```text
enabled
```

Check status.

```bash
systemctl status start-containers.service
```

---

# 6. Restore Cron Jobs

Edit root's crontab.

```bash
crontab -e
```

Add:

```cron
30 0 * * * /root/scripts/restart-proxmox.sh
```

Verify.

```bash
crontab -l
```

Expected

```cron
30 0 * * * /root/scripts/restart-proxmox.sh
```

---

# 7. Reboot

```bash
reboot
```

---

# 8. Verify Startup Orchestration

Check service.

```bash
systemctl status start-containers.service
```

Verify log.

```bash
cat /var/log/start-containers.log
```

or

```bash
tail -f /var/log/start-containers.log
```

Expected

- Startup orchestration started
- Operating hours or Off-hours detected
- Containers processed
- Startup orchestration finished

---

# 9. Verify Containers

```bash
pct list
```

Expected

```text
100 running
101 running
102 running
```

---

# 10. Verify Services

## Pi-hole

```text
http://192.168.1.10/admin
```

DNS test

```bash
nslookup google.com 192.168.1.10
```

---

## WireGuard

```bash
pct enter 101
wg show
```

Verify

- Interface up
- Listening port 51820
- Peers listed

---

## Uptime Kuma

```text
http://192.168.1.30:3001
```

Verify all monitors are online.

---

# 11. Router Verification

Router

```text
192.168.1.1
```

Verify

- DNS = 192.168.1.10
- UDP 51820 forwarded to 192.168.1.20
- DHCP enabled
- IPv6 DHCP disabled
- IPv6 RA disabled

---

# 12. Final Checks

```bash
pct list

systemctl is-enabled start-containers.service

crontab -l

cat /var/log/start-containers.log

wg show
```

All containers should be running and all services operational.
