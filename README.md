# Homelab

## Hardware

- Intel NUC7i3DNB
- 8 GB RAM
- 128 GB NVMe

## Network

Internet
↓
FiberHome HG6544C (ONT)
↓
ZTE ZXHN H3601P
↓
Proxmox

## Containers

### CT100 - Pi-hole

IP: 192.168.1.10

Purpose:
- Network DNS
- Ad blocking

Services:
- Pi-hole

---

### CT101 - WireGuard

IP: 192.168.1.20

Purpose:
- VPN

Services:
- WireGuard

---

### CT102 - Uptime Kuma

IP: 192.168.1.30

Purpose:
- Monitoring

Checks:
- Internet
- Pi-hole
- WireGuard

---

### CT103 - Reserved

Purpose:
- Future Expenses application

Status:
- Not functional

---

## Backups

Weekly:
- Proxmox vzdump
- Copy to laptop

## Port Forwarding

UDP 51820 → 192.168.1.20

## DNS

Primary:
192.168.1.10 (Pi-hole)

Secondary:
None

## Future

- Jellyfin
- Expenses
