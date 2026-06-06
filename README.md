# 🍯 Cowrie Honeypot — Docker Compose - composneypot

A docker compose for cowrie honeypot
Honeypot SSH/Telnet based on [Cowrie](https://github.com/cowrie/cowrie).

---

## Files

```
.
├── docker-compose.yml       ← docker Orchestration
├── .env                     ← ports, version, timezone (copy of .env.example)
├── .env.example
├── cowrie.cfg.example       ← cowrie internal configuration template
└── cowrie/
    ├── etc/
    │   ├── cowrie.cfg       ← config active (copy of cowrie.cfg.example)
    │   └── userdb.txt       ← accepted credentials (optional)
    └── var/
        ├── log/             ← cowrie.json, cowrie.log
        ├── downloads/       ← files uploaded by attackers
        └── tty/             ← recorded sessions
```

**General rule:**
- `.env` → docker parameters (host ports, restart policy...)
- `cowrie/etc/cowrie.cfg` → cowrie internal configuration (hostname, telnet, sensor_name...)

---

## Startup

```bash
# 1. Docker variables
cp .env.example .env

# 2. Cowrie configuration
mkdir -p cowrie/etc cowrie/var/log cowrie/var/downloads cowrie/var/tty
cp cowrie.cfg.example cowrie/etc/cowrie.cfg
nano cowrie/etc/cowrie.cfg

# 3. Permissions (cowrie runs with uid/gid 999 in the container)
sudo chown -R 999:999 cowrie/var/

# 4. Up
docker compose up -d
docker compose logs -f
```

---

## Quick setup (cowrie.cfg)

```ini
[honeypot]
hostname = myserver       # attacker prompt: root@myserver:~#
sensor_name = my-honeypot   # "sensor" field in cowrie.json

[telnet]
enabled = yes               # enable Telnet in addition to SSH
```

## Accepted credentials (userdb.txt)

```
# user:uid:password  (* = any password)
root:0:*
admin:0:admin
```

---

## See logs in real time

```bash
# Parseed JSON
tail -f cowrie/var/log/cowrie.json | python3 -m json.tool

# Play recorded session
docker exec -it cowrie playlog var/lib/cowrie/tty/<fichero>.tty
```

---

> Made with 🍯 by [bursao](https://github.com/bursao)

---

## ⚠️ warnings

- Isolated network deployment or DMZ.
- If the host uses port 22 for real SSH, move it to another port first in `/etc/ssh/sshd_config`.
- Check `cowrie/var/downloads/` — may contain real malware.