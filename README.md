# Orange Pi 5 Home Assistant Hub

Complete guide to setting up a smart home hub on Orange Pi 5 with WiFi/BT module.

## 🏠 What You'll Build

| Service | Description |
|---------|-------------|
| Home Assistant | Core automation platform |
| Frigate | NVR with local AI detection |
| Jellyfin | Media streaming server |
| Mosquitto | MQTT broker (optional) |

## 📁 Guide Files

| File | Description |
|------|-------------|
| [part0-orange-pi-setup.md](part0-orange-pi-setup.md) | Flash image + enable WiFi/BT |
| [part1-home-assistant.md](part1-home-assistant.md) | Docker HA setup + HACS |
| [part2-frigate.md](part2-frigate.md) | NVR + camera setup |
| [part3-media-voice.md](part3-media-voice.md) | Jellyfin media server |

## 🔧 Hardware Required

- Orange Pi 5 (8GB) 
- microSD card 16GB+ (Class 10) (Mine one is 64GB got from Facebook Market Place ~ $10)
- AP6275P M.2 WiFi 6 + BT 5.0 module
- Power supply (5V 4A USB-C) (~$20)
- Camera(s) for Frigate (optional) (~$80)

## 🚀 Quick Start

1. Flash Ubuntu image to microSD ([Part 0](part0-orange-pi-setup.md))
2. Install Home Assistant via Docker ([Part 1](part1-home-assistant.md))
3. Set up Frigate NVR ([Part 2](part2-frigate.md))
4. Add Jellyfin media server ([Part 3](part3-media-voice.md))

## 📊 Architecture

```
┌─────────────────────────────────────────────────────────────┐
│                    Orange Pi 5                              │
├─────────────────────────────────────────────────────────────┤
│  Docker                                                      │
│  ┌──────────────┐ ┌──────────────┐ ┌──────────────┐        │
│  │ Home         │ │ Frigate      │ │ Jellyfin     │        │
│  │ Assistant    │ │ (NVR)        │ │ (Media)      │        │
│  │ :8123       │ │ :8971        │ │ :8096        │        │
│  └──────────────┘ └──────────────┘ └──────────────┘        │
│                                                              │
│  AP6275P: WiFi 6 + Bluetooth 5.0                           │
└─────────────────────────────────────────────────────────────┘
```

## ⚠️ Notes

- Replace `<YOUR_ORANGE_PI_IP>` with your actual IP address
- Replace `<YOUR_CAMERA_IP>` with your camera's IP address
- All passwords should be changed from defaults

## 📜 License

MIT License - Use freely for personal projects.

---

*Created: February 2026*
