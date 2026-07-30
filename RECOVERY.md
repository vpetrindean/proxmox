# RECOVERY.md

# Disaster Recovery Guide

This document describes the complete recovery procedure for rebuilding
the homelab after a total Proxmox host failure.

------------------------------------------------------------------------

# Prerequisites

Required:

-   Replacement hardware (or repaired Intel NUC)
-   Proxmox VE installation media
-   Latest LXC backups
-   Backup copies of:
    -   `/root/scripts/`
    -   `start-containers.service`
    -   `README.md`
    -   `RECOVERY.md`

------------------------------------------------------------------------

# Network Configuration

Hostname

``` text
proxmox
```

Management IP

``` text
192.168.1.50/24
```

Gateway

``` text
192.168.1.1
```

DNS

``` text
192.168.1.10
```

Bridge

``` text
vmbr0
```

Timezone

``` text
Europe/Bucharest
```

------------------------------------------------------------------------

# Install Proxmox

Install Proxmox VE.

After installation verify:

``` bash
hostnamectl
ip addr
ip route
```

------------------------------------------------------------------------

# Copy Backups

Destination:

``` text
/var/lib/vz/dump/
```

Example:

``` bash
scp *.tar.zst root@192.168.1.50:/var/lib/vz/dump/
```

Verify:

``` bash
ls -lh /var/lib/vz/dump
```

------------------------------------------------------------------------

# Restore Containers

Restore Pi-hole

``` bash
pct restore 100 /var/lib/vz/dump/vzdump-lxc-100-<date>.tar.zst local-lvm
```

Restore WireGuard

``` bash
pct restore 101 /var/lib/vz/dump/vzdump-lxc-101-<date>.tar.zst local-lvm
```

Restore Uptime Kuma

``` bash
pct restore 102 /var/lib/vz/dump/vzdump-lxc-102-<date>.tar.zst local-lvm
```

Verify

``` bash
pct list
```

Expected:

``` text
100 stopped
101 stopped
102 stopped
```

Do not manually create containers before restoring.

------------------------------------------------------------------------

# Restore Scripts

Create directory:

``` bash
mkdir -p /root/scripts
```

Copy:

``` text
restart-proxmox.sh
shutdown-proxmox.sh
start-containers.sh
```

Verify:

``` bash
ls -l /root/scripts
```

Set permissions:

``` bash
chmod +x /root/scripts/*.sh
```

------------------------------------------------------------------------

# Restore systemd Service

Copy:

``` text
start-containers.service
```

to

``` text
/etc/systemd/system/start-containers.service
```

Reload:

``` bash
systemctl daemon-reload
```

Enable:

``` bash
systemctl enable start-containers.service
```

Verify:

``` bash
systemctl is-enabled start-containers.service
systemctl status start-containers.service
```

------------------------------------------------------------------------

# Restore Cron

Edit root crontab:

``` bash
crontab -e
```

Add:

``` cron
30 0 * * * /root/scripts/restart-proxmox.sh
```

Verify:

``` bash
crontab -l
```

------------------------------------------------------------------------

# First Boot

Reboot:

``` bash
reboot
```

------------------------------------------------------------------------

# Verify Startup

Startup log

``` bash
cat /var/log/start-containers.log
```

Live

``` bash
tail -f /var/log/start-containers.log
```

Container status

``` bash
pct list
```

Expected:

``` text
100 running
101 running
102 running
```

------------------------------------------------------------------------

# Verify Pi-hole (also check if the lists are there)

Open:

``` text
http://192.168.1.10/admin
```

DNS test:

``` bash
nslookup google.com 192.168.1.10
```

------------------------------------------------------------------------

# Verify WireGuard

``` bash
pct enter 101
wg show
```

Verify:

-   Interface present
-   Listening port 51820
-   Peers listed

------------------------------------------------------------------------

# Verify Uptime Kuma

Open:

``` text
http://192.168.1.30:3001
```

Ensure monitors are online.

------------------------------------------------------------------------

# Verify Router

Router IP

``` text
192.168.1.1
```

Confirm:

-   DHCP enabled
-   DNS = 192.168.1.10
-   DHCPv6 disabled
-   RA disabled
-   UDP 51820 forwarded to 192.168.1.20

------------------------------------------------------------------------

# Final Validation

Run:

``` bash
pct list
systemctl is-enabled start-containers.service
systemctl status start-containers.service
crontab -l
cat /var/log/start-containers.log

pct enter 101
wg show
```

Expected outcome:

-   All containers running
-   Startup service enabled
-   Cron present
-   WireGuard operational
-   Pi-hole reachable
-   Uptime Kuma reachable

Recovery complete.

☐ Proxmox reachable at 192.168.1.50

☐ CT100 running

☐ CT101 running

☐ CT102 running

☐ DNS working

☐ Internet reachable

☐ Pi-hole admin reachable

☐ Gravity updated

☐ WireGuard handshake successful

☐ Uptime Kuma reachable

☐ Port forwarding verified

☐ Router DNS = 192.168.1.10

☐ Startup service enabled

☐ Cron present

☐ start-containers.log contains no errors
