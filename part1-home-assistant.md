# Part 1: Home Assistant Installation via Docker

**Time estimate:** 30-60 minutes  
**Difficulty:** Beginner  
**Status:** ✅ Complete

---

## Overview

We'll install Home Assistant Container using Docker. This gives us flexibility and keeps the system clean.

> ⚠️ Note: Home Assistant Container (Docker) has some limitations vs HAOS:
> - No add-on store (we'll run services as separate containers)
> - Manual backup management
> - But: More control, less overhead

---

## Step 1.1: Prepare Directories ✅

**Executed on Orange Pi 5:**

```bash
# Create Home Assistant config directory
sudo mkdir -p /opt/homeassistant/config
sudo chown -R $USER:$USER /opt/homeassistant

# Create Docker network for HA services
docker network create ha-network
```

**Result:**
```
/opt/homeassistant/
└── config/     # Home Assistant config files
```

---

## Step 1.2: Create Docker Compose File ✅

**Executed on Orange Pi 5:**

```bash
cat > /opt/homeassistant/docker-compose.yml << "EOF"
services:
  homeassistant:
    image: ghcr.io/home-assistant/home-assistant:stable
    container_name: homeassistant
    restart: unless-stopped
    privileged: true
    network_mode: host
    volumes:
      - ./config:/config
      - /etc/localtime:/etc/localtime:ro
      - /run/dbus:/run/dbus:ro
    environment:
      - TZ=Australia/Sydney
EOF
```

**File:** `/opt/homeassistant/docker-compose.yml`

**Note:** The `version: "3.8"` line is obsolete in modern Docker Compose.

---

## Step 1.3: Start Home Assistant ✅

**Executed on Orange Pi 5:**

```bash
cd /opt/homeassistant

# Pull and start (image is ~1GB)
docker compose pull
docker compose up -d

# Check logs
docker logs -f homeassistant
```

**Result:**
```
CONTAINER ID   IMAGE                                          COMMAND
4011bbe4781b   ghcr.io/home-assistant/home-assistant:stable   "/init"
```

---

## Step 1.4: Initial Setup ✅

### Access Home Assistant

Open in browser: **http://<YOUR_ORANGE_PI_IP>:8123**

### Onboarding Steps

1. **Create Account** - Set your name, username, password
2. **Location Settings** - Set your home location, Metric, AUD
3. **Analytics & Updates** - Your choice
4. **Skip Integrations** - We'll add devices later

---

## Step 1.5: Install HACS ✅

**HACS** (Home Assistant Community Store) gives access to custom integrations and themes.

### Run in Terminal

```bash
# Install HACS inside the container
docker exec homeassistant bash -c "wget -O - https://get.hacs.xyz | bash -"

# Restart Home Assistant
docker compose restart
```

**Output:**
```
INFO: Found Home Assistant configuration directory at '/config'
INFO: Creating custom_components directory...
INFO: Downloading HACS
INFO: Unpacking HACS...
INFO: Installation complete.
```

### Enable in Home Assistant

After installation, you need to configure HACS in Home Assistant:

1. Open **http://<YOUR_ORANGE_PI_IP>:8123**
2. Go to **Settings → Devices & Services**
3. Click **"+ Add Integration"**
4. Search for **"HACS"**
5. Follow the setup wizard

**📖 Full setup guide:** https://www.hacs.xyz/docs/use/configuration/basic/#to-set-up-the-hacs-integration

> **Note:** HACS requires a GitHub account to link during setup.

---

## Step 1.6: Configure Backups ⬜ Optional

1. Settings → System → Backups
2. Enable **"Automatic backups"**
3. Set schedule: Daily at 3:00 AM
4. Set backup location (external drive recommended)

---

## Useful Commands

```bash
# Start Home Assistant
cd /opt/homeassistant && docker compose up -d

# View logs (follow mode)
docker logs -f homeassistant

# View recent logs
docker logs --tail 100 homeassistant

# Restart container
docker compose restart

# Stop container
docker compose down

# Check container status
docker ps

# Check resource usage
docker stats homeassistant
```

---

## Verification Checklist

| Item | Status |
|------|--------|
| Home Assistant accessible | ✅ |
| Admin account created | ✅ |
| Location configured | ✅ |
| HACS installed | ✅ |
| Backup configured | ⬜ |

---

## Common Issues

### Port 8123 already in use
```bash
sudo lsof -i :8123
```

### Permission denied on volumes
```bash
sudo chown -R $USER:$USER /opt/homeassistant
```

### Container won't start
```bash
docker compose logs homeassistant
```

### Can't access web UI
```bash
# Check if container is running
docker ps

# Check if port is listening
netstat -tlnp | grep 8123
```

---

## Next Steps

Continue to [Part 2: Frigate NVR](./part2-frigate.md) for camera setup.

---

## Progress Log

| Date | Step | Status |
|------|------|--------|
| 2026-02-27 21:07 | 1.1 - Create directories | ✅ Complete |
| 2026-02-27 21:09 | 1.2 - Create docker-compose.yml | ✅ Complete |
| 2026-02-27 21:09 | 1.3 - Pull image (1GB) | ✅ Complete |
| 2026-02-27 21:12 | 1.3 - Start container | ✅ Complete |
| 2026-02-27 21:14 | 1.4 - Web UI accessible | ✅ Complete |
| 2026-02-27 21:19 | 1.4 - Onboarding | ✅ Complete |
| 2026-02-27 21:23 | 1.5 - HACS installed | ✅ Complete |
| 2026-02-27 21:39 | 1.7 - HACS needs config in HA UI | 📖 [Setup Guide](https://www.hacs.xyz/docs/use/configuration/basic/#to-set-up-the-hacs-integration) |

---

*Part 1 - Home Assistant Installation*  
*Last updated: 2026-02-27 21:39 GMT+11*