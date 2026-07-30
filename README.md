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

# Host Recovery

1. Install Proxmox VE.
2. Set hostname to `proxmox`.
3. Configure:
   - IP: 192.168.1.50/24
   - Gateway: 192.168.1.1
   - DNS: 192.168.1.10
4. Create `vmbr0`.
5. Restore LXC backups.
6. Copy `/root/scripts`.
7. Restore root crontab.
8. Verify:
   - Pi-hole
   - WireGuard
   - Uptime Kuma
