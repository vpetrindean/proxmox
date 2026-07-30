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

## Backup Location

Copy all LXC backup files to the default Proxmox backup directory.

Destination:

```text
/var/lib/vz/dump/
```

Example:

```bash
scp *.tar.zst root@192.168.1.50:/var/lib/vz/dump/
```

or

```bash
cp /mnt/usb/*.tar.zst /var/lib/vz/dump/
```

Verify:

```bash
ls -lh /var/lib/vz/dump
```

Expected:

```text
vzdump-lxc-100-YYYY_MM_DD-HH_MM_SS.tar.zst
vzdump-lxc-101-YYYY_MM_DD-HH_MM_SS.tar.zst
vzdump-lxc-102-YYYY_MM_DD-HH_MM_SS.tar.zst
```

---

## Restore CT100 (Pi-hole)

Restore:

```bash
pct restore 100 \
/var/lib/vz/dump/vzdump-lxc-100-YYYY_MM_DD-HH_MM_SS.tar.zst \
local-lvm
```

Wait until the restore finishes.

Verify:

```bash
pct list
```

Expected:

```text
100 stopped
```

---

## Restore CT101 (WireGuard)

```bash
pct restore 101 \
/var/lib/vz/dump/vzdump-lxc-101-YYYY_MM_DD-HH_MM_SS.tar.zst \
local-lvm
```

---

## Restore CT102 (Uptime Kuma)

```bash
pct restore 102 \
/var/lib/vz/dump/vzdump-lxc-102-YYYY_MM_DD-HH_MM_SS.tar.zst \
local-lvm
```

---

## Restore CT103 (Expenses)

Only if a backup exists.

```bash
pct restore 103 \
/var/lib/vz/dump/vzdump-lxc-103-YYYY_MM_DD-HH_MM_SS.tar.zst \
local-lvm
```

---

## Verify Containers

```bash
pct list
```

Expected:

```text
100 stopped
101 stopped
102 stopped
103 stopped
```

Do **not** start them manually.

The startup orchestration service starts them automatically after the host boots.

---

## Verify Startup Configuration

Each restored container should keep:

- Container ID
- Hostname
- Static IP
- Mount points
- Installed software
- Configuration
- Startup options
- Network configuration

Verify:

```bash
pct config 100
pct config 101
pct config 102
```

Check:

```text
net0:
onboot:
startup:
```

---

## Restore Complete

Reboot the Proxmox host.

```bash
reboot
```

The following sequence should occur automatically:

```text
Proxmox boots
        │
        ▼
start-containers.service
        │
        ▼
start-containers.sh
        │
        ▼
CT100 starts
        │
30 s delay
        │
▼
CT101 starts
        │
30 s delay
        │
▼
CT102 starts
        │
30 s delay
        │
▼
CT103 starts
```

---

## Verify Startup

Container status

```bash
pct list
```

Startup log

```bash
cat /var/log/start-containers.log
```

Live monitoring

```bash
tail -f /var/log/start-containers.log
```

WireGuard

```bash
pct enter 101
wg show
```

Pi-hole

```text
http://192.168.1.10/admin
```

Uptime Kuma

```text
http://192.168.1.30:3001
```

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
