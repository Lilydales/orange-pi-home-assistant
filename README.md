# Orange Pi 5 Home Assistant Hub Guide

**Hardware:** Orange Pi 5 + AP6275P WiFi 6/BT 5.0 module  
**OS:** Ubuntu Jammy Server (Linux 6.1.99)  
**Image:** Orangepi5_1.2.2_ubuntu_jammy_server_linux6.1.99  
**Goal:** High-performance Home Assistant hub with local AI processing

> **Context:** Orange Pi won via contest - must run all services on this device

---

## Overview

This guide covers building a fully local smart home hub with:

| Part | Service | Description | Status |
|------|---------|-------------|--------|
| 0 | Orange Pi Setup | Flash image + enable WiFi/BT | ✅ Complete |
| 1 | Home Assistant | Core automation platform | ✅ Complete |
| 2 | Frigate | NVR + object detection | ✅ Complete |
| 3 | Media + Voice | Jellyfin + Voice Assistant | ✅ Complete |

---

## Hardware Required

- Orange Pi 5 (8GB) 
- microSD card 16GB+ (Class 10) (Mine is 64GB from Facebook Marketplace ~$10)
- AP6275P M.2 WiFi 6 + BT 5.0 module
- Power supply (5V 4A USB-C) (~$20)
- Camera(s) for Frigate (optional) (~$80)

---

## Architecture

```
┌─────────────────────────────────────────────────────────────┐
│                    Orange Pi 5                              │
│                  (8GB RAM)                                  │
│                  IP: <YOUR_ORANGE_PI_IP>                    │
├─────────────────────────────────────────────────────────────┤
│  Docker                                                      │
│  ┌──────────────┐ ┌──────────────┐ ┌──────────────┐        │
│  │ Home         │ │ Frigate      │ │ Jellyfin     │        │
│  │ Assistant    │ │ (NVR)        │ │ (Media)      │        │
│  │ :8123       │ │ :8971        │ │ :8096        │        │
│  └──────────────┘ └──────────────┘ └──────────────┘        │
│                                                              │
│  ┌──────────────────────────────────────────────────────┐  │
│  │ Voice: Whisper (STT) + Piper (TTS) + openWakeWord   │  │
│  │ :10200 (STT)  :10201 (TTS)  :10400 (Wake Word)      │  │
│  └──────────────────────────────────────────────────────┘  │
│                                                              │
│  AP6275P: WiFi 6 + Bluetooth 5.0                           │
└─────────────────────────────────────────────────────────────┘
```

---

## Part 0: Orange Pi 5 Initial Setup

**Status:** ✅ Complete

### What Was Done

| Step | Description | Status |
|------|-------------|--------|
| 1 | Download & extract image | ✅ |
| 2 | Flash to microSD with Raspberry Pi Imager | ✅ |
| 3 | Install AP6275P module (bottom M.2 slot) | ✅ |
| 4 | Enable WiFi via `orangepi-config` | ✅ |
| 5 | Connect to WiFi | ✅ |
| 6 | Enable SSH | ✅ |

### Connection Info

| Item | Value |
|------|-------|
| IP Address | `<YOUR_ORANGE_PI_IP>` |
| Username | `orangepi` |
| SSH Command | `ssh orangepi@<YOUR_ORANGE_PI_IP>` |
| OS | Ubuntu Jammy Server (Linux 6.1.99) |
| Docker | 29.2.1 |

📖 Full details: [Part 0 - Orange Pi Setup](./part0-orange-pi-setup.md)

---

## Part 1: Home Assistant Installation

**Status:** ✅ Complete

### What Was Done

| Step | Description | Status |
|------|-------------|--------|
| 1.1 | Create directories (`/opt/homeassistant/config`) | ✅ |
| 1.2 | Create Docker network (`ha-network`) | ✅ |
| 1.3 | Create `docker-compose.yml` | ✅ |
| 1.4 | Pull Home Assistant image (~1GB) | ✅ |
| 1.5 | Start container | ✅ |
| 1.6 | Web UI onboarding | ✅ |
| 1.7 | Install HACS | ✅ |
| 1.8 | Configure HACS in HA | ⬜ [Setup Guide](https://www.hacs.xyz/docs/use/configuration/basic/#to-set-up-the-hacs-integration) |

### Docker Compose File

**Location:** `/opt/homeassistant/docker-compose.yml`

```yaml
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
```

### Access

- **URL:** http://<YOUR_ORANGE_PI_IP>:8123

📖 Full details: [Part 1 - Home Assistant](./part1-home-assistant.md)

---

## Part 2: Frigate NVR

**Status:** ✅ Complete

### What Was Done

| Step | Description | Status |
|------|-------------|--------|
| 2.1 | Create directories (`/opt/frigate/`) | ✅ |
| 2.2 | Create `config.yml` | ✅ |
| 2.3 | Create `docker-compose.yml` | ✅ |
| 2.4 | Pull Frigate image (~900MB) | ✅ |
| 2.5 | Start container | ✅ |
| 2.6 | Install Mosquitto MQTT | ✅ (Optional) |
| 2.7 | Add living room camera | ✅ |
| 2.8 | Frigate credentials (auto-generated) | ✅ |
| 2.9 | Add HA integration via HACS | ✅ |
| 2.10 | Enable WebRTC in HA Frigate config | ✅ |

> **MQTT is optional.** Cameras work via API. MQTT only needed for detection events/automations.

### Docker Compose File

**Location:** `/opt/frigate/docker-compose.yml`

```yaml
services:
  frigate:
    image: ghcr.io/blakeblackshear/frigate:stable
    container_name: frigate
    restart: unless-stopped
    privileged: true
    shm_size: "256mb"
    devices:
      - /dev/dri:/dev/dri
    volumes:
      - /etc/localtime:/etc/localtime:ro
      - ./config:/config
      - ./media:/media/frigate
    ports:
      - "8971:8971"
      - "8554:8554"
      - "8555:8555"
      - "8555:8555/udp"
    environment:
      - TZ=Australia/Sydney
```

### Access

- **URL:** https://<YOUR_ORANGE_PI_IP>:8971
- **Detection:** CPU-based (no Coral required)
- **Username:** `admin`
- **Password:** Auto-generated (see logs: `docker logs frigate 2>&1 | grep -A1 "User:"`)

> **Note:** This is the Docker version of Frigate. When adding to Home Assistant, enter the IP address and login credentials manually.

### Enable Camera in Home Assistant

After adding the Frigate integration:

1. **Settings → Devices & Services → Frigate → Configure**
2. ✅ Tick **"Use Frigate-native WebRTC support"**
3. Submit - the camera will now appear in HA

📖 Full details: [Part 2 - Frigate](./part2-frigate.md)

---

## Part 3: Media Server & Voice Assistant

**Status:** ✅ Complete

### What Was Done

| Step | Description | Status |
|------|-------------|--------|
| 3.1 | Jellyfin media server | ✅ |
| 3.2 | Jellyfin HA integration | ✅ |
| 3.3 | Whisper speech-to-text (STT) | ✅ Port 10200 |
| 3.4 | Piper text-to-speech (TTS) | ✅ Port 10201 |
| 3.5 | openWakeWord detection | ✅ Port 10400 |
| 3.6 | Configure HA voice pipeline | ⬜ |

### Voice Services

| Service | Port | Purpose |
|---------|------|---------|
| Whisper | 10200 | Speech-to-text |
| Piper | 10201 | Text-to-speech |
| openWakeWord | 10400 | Wake word ("Hey Jarvis") |

### Access

- **Jellyfin:** http://<YOUR_ORANGE_PI_IP>:8096
- **Voice:** All local, no cloud required!

📖 Full details: [Part 3 - Media & Voice](./part3-media-voice.md)

---

## Quick Reference

| Service | Port | URL |
|---------|------|-----|
| SSH | 22 | `ssh orangepi@<YOUR_ORANGE_PI_IP>` |
| Home Assistant | 8123 | http://<YOUR_ORANGE_PI_IP>:8123 |
| Frigate | 8971 | https://<YOUR_ORANGE_PI_IP>:8971 |
| Mosquitto MQTT | 1883 | mqtt://<YOUR_ORANGE_PI_IP>:1883 (optional) |
| Jellyfin | 8096 | http://<YOUR_ORANGE_PI_IP>:8096 |
| Whisper (STT) | 10200 | http://<YOUR_ORANGE_PI_IP>:10200 |
| Piper (TTS) | 10201 | http://<YOUR_ORANGE_PI_IP>:10201 |
| openWakeWord | 10400 | http://<YOUR_ORANGE_PI_IP>:10400 |

---

## Useful Commands

```bash
# Check all containers
docker ps

# View logs
docker logs -f homeassistant
docker logs -f frigate
docker logs -f whisper
docker logs -f piper
docker logs -f openwakeword

# Restart services
cd /opt/homeassistant && docker compose restart
cd /opt/frigate && docker compose restart
cd /opt/wyoming && docker compose restart

# Check memory
free -h

# Check disk
df -h
```

---

## System Resources

| Resource | Used | Total |
|----------|------|-------|
| RAM | ~2GB | 7.7GB |
| Disk | 15GB | 58GB |
| Containers | 7 | homeassistant, frigate, mosquitto, jellyfin, whisper, piper, openwakeword |

---

## Progress Log

| Date | Progress |
|------|----------|
| 2026-02-27 21:07 | Part 0 complete - Orange Pi flashed, WiFi enabled |
| 2026-02-27 21:12 | Part 1 - Home Assistant container running |
| 2026-02-27 21:19 | Part 1 - Web UI onboarding complete |
| 2026-02-27 21:23 | Part 1 - HACS installed |
| 2026-02-27 21:35 | Part 2 - Frigate container running (healthy) |
| 2026-02-27 21:54 | Guide updated - Frigate login & HA integration notes |
| 2026-02-27 22:06 | Part 2 - Mosquitto MQTT installed (optional) |
| 2026-02-27 22:07 | Part 2 - Living room camera added |
| 2026-02-27 22:23 | Part 2 - WebRTC tip for HA camera visibility |
| 2026-02-27 22:27 | Part 3 - Jellyfin installed and running |
| 2026-02-27 22:38 | Guide updated - MQTT optional, Jellyfin HA integration |
| 2026-02-27 23:03 | Part 3 - Voice assistant (Whisper, Piper, openWakeWord) |
| 2026-02-27 23:24 | Fixed Wyoming services (TCP binding issue) |

---

*Guide created: February 2026*  
*Last updated: 2026-02-27 23:24 GMT+11*