# Scheduled Scripts

Scripts location:

```text
/root/scripts
```

Cron:

```cron
30 0 * * * /root/scripts/restart-proxmox.sh
```

---

## restart-proxmox.sh

Location

```text
/root/scripts/restart-proxmox.sh
```

Purpose

- Gracefully shuts down all LXC containers.
- Waits 10 seconds.
- Reboots the Proxmox host. :contentReference[oaicite:0]{index=0}

Shutdown order

```text
CT103
CT102
CT101
CT100
```

Dependencies

- `pct`
- `reboot`

Verification

```bash
last reboot
```

If something goes wrong

```bash
journalctl -b -1
journalctl -u pveproxy
journalctl -u pvedaemon
```

---

## shutdown-proxmox.sh

Location

```text
/root/scripts/shutdown-proxmox.sh
```

Purpose

- Gracefully shuts down all LXC containers.
- Waits 10 seconds.
- Powers off the Proxmox host. :contentReference[oaicite:1]{index=1}

Shutdown order

```text
CT103
CT102
CT101
CT100
```

Dependencies

- `pct`
- `shutdown`

Verification

```bash
last -x
```

If something goes wrong

```bash
journalctl -b -1
journalctl -u pveproxy
journalctl -u pvedaemon
```

---

## start-containers.sh

Location

```text
/root/scripts/start-containers.sh
```

Purpose

Controls the startup sequence after every Proxmox boot. :contentReference[oaicite:2]{index=2}

Logic

- Creates a new log file.
- Detects current time.
- During off-hours (00:00–05:59):
  - Waits up to 240 seconds for a successful Proxmox web login.
  - If no login occurs, executes `shutdown-proxmox.sh`.
  - If a login occurs, continues normal startup.
- During operating hours:
  - Waits 300 seconds after boot.
  - Starts containers sequentially.
  - Waits 30 seconds between each container.

Container startup order

```text
CT100
CT101
CT102
CT103
```

Dependencies

- `pct`
- `journalctl`
- `grep`
- `tee`
- `/root/scripts/shutdown-proxmox.sh`

Log file

```text
/var/log/start-containers.log
```

Monitor log live

```bash
tail -f /var/log/start-containers.log
```

Verify today's execution

```bash
cat /var/log/start-containers.log
```

If something goes wrong

```bash
cat /var/log/start-containers.log
journalctl -b
journalctl -u pvedaemon
journalctl -u pveproxy
```
