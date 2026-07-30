# Homelab

## Hardware

- Intel NUC7i3DNB
- Intel Core i3-7100U
- 8 GB DDR4 RAM
- 128 GB NVMe SSD
- Proxmox VE

---

# Network

```
Internet
    │
FiberHome HG6544C (ONT)
    │
ZTE ZXHN H3601P
    │
Proxmox
```

LAN:

```
192.168.1.0/24
```

Router:

```
192.168.1.1
```

---

# Running Containers

## CT100 - Pi-hole

**IP**

```
192.168.1.10
```

### Purpose

- Network-wide DNS
- Ad blocking

### Services

- Pi-hole

---

## CT101 - WireGuard

**IP**

```
192.168.1.20
```

### Purpose

- Remote VPN access

### Services

- WireGuard

### Port Forwarding

```
UDP 51820
```

---

## CT102 - Uptime Kuma

**IP**

```
192.168.1.30
```

### Purpose

- Service monitoring

### Monitors

- Internet
- Pi-hole
- WireGuard

---

# DNS

Primary DNS:

```
192.168.1.10
```

Secondary DNS:

```
None
```

---

# IPv6

Current status:

```
Enabled on router
```

(Adjust this section if you later decide to permanently disable IPv6.)

---

# Backup Strategy

Weekly:

- Proxmox LXC backups (`vzdump`)
- Copy backups to laptop

---

# Maintenance

After every Proxmox update:

- Backup all containers
- Verify Pi-hole
- Verify WireGuard
- Verify Uptime Kuma

---

# Port Forwarding

| Service | Protocol | Port | Destination |
|----------|----------|-----:|-------------|
| WireGuard | UDP | 51820 | 192.168.1.20 |

---

# Services Summary

| Container | Purpose | Status |
|-----------|---------|--------|
| CT100 | Pi-hole | ✅ Running |
| CT101 | WireGuard | ✅ Running |
| CT102 | Uptime Kuma | ✅ Running |
