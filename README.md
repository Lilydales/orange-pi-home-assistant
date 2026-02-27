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
| 1 | Home Assistant | Core automation platform | ✅ Running |
| 2 | Frigate | NVR + object detection | ✅ Running |
| 3 | Media + Voice | Jellyfin + Voice Assistant | ✅ Running |

## Hardware Required

- Orange Pi 5 (8GB) 
- microSD card 16GB+ (Class 10) (Mine is 64GB from Facebook Marketplace ~$10)
- AP6275P M.2 WiFi 6 + BT 5.0 module
- Power supply (5V 4A USB-C) (~$20)
- Camera(s) for Frigate (optional) (~$80)

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

## Part 0: Orange Pi 5 Initial Setup

**Status:** ✅ Complete

| Item | Value |
|------|-------|
| IP Address | `<YOUR_ORANGE_PI_IP>` |
| Username | `orangepi` |
| SSH Command | `ssh orangepi@<YOUR_ORANGE_PI_IP>` |

📖 [Part 0 - Orange Pi Setup](part0-orange-pi-setup.md)

## Part 1: Home Assistant Installation

**Status:** ✅ Running

| Step | Description | Status |
|------|-------------|--------|
| 1.1 | Create directories | ✅ |
| 1.2 | Docker compose setup | ✅ |
| 1.3 | HACS installed | ✅ |

📖 [Part 1 - Home Assistant](part1-home-assistant.md)

## Part 2: Frigate NVR

**Status:** ✅ Running

| Service | Port | Status |
|---------|------|--------|
| Frigate | 8971 | ✅ |
| Mosquitto MQTT | 1883 | ✅ (optional) |

> **MQTT is optional** - Cameras work via API. MQTT only needed for detection events/automations.

📖 [Part 2 - Frigate](part2-frigate.md)

## Part 3: Media Server & Voice Assistant

**Status:** ✅ Running

### Services

| Service | Port | Purpose |
|---------|------|---------|
| Jellyfin | 8096 | Media streaming |
| Whisper | 10200 | Speech-to-text |
| Piper | 10201 | Text-to-speech |
| openWakeWord | 10400 | Wake word detection |

All voice processing runs **locally** - no cloud required!

📖 [Part 3 - Media & Voice](part3-media-voice.md)

## Quick Reference

| Service | Port | URL |
|---------|------|-----|
| SSH | 22 | `ssh orangepi@<YOUR_ORANGE_PI_IP>` |
| Home Assistant | 8123 | `http://<YOUR_ORANGE_PI_IP>:8123` |
| Frigate | 8971 | `https://<YOUR_ORANGE_PI_IP>:8971` |
| Jellyfin | 8096 | `http://<YOUR_ORANGE_PI_IP>:8096` |
| Whisper (STT) | 10200 | `http://<YOUR_ORANGE_PI_IP>:10200` |
| Piper (TTS) | 10201 | `http://<YOUR_ORANGE_PI_IP>:10201` |
| openWakeWord | 10400 | `http://<YOUR_ORANGE_PI_IP>:10400` |

---

*Guide created: February 2026*  
*Last updated: 2026-02-27*
