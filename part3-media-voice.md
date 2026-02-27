# Part 3: Media Server & Voice Assistant

**Time estimate:** 30-60 minutes  
**Difficulty:** Intermediate  
**Status:** 🔄 In Progress

---

## Overview

This part covers:
- **Jellyfin** - Free media streaming server
- **Whisper** - Local speech-to-text (STT)
- **Piper** - Local text-to-speech (TTS)
- **openWakeWord** - Wake word detection

---

## Section A: Jellyfin Media Server ✅

### Quick Access

- **URL:** http://<YOUR_ORANGE_PI_IP>:8096
- **Status:** Running ✅

### Setup Steps

1. Open Jellyfin in browser
2. Create admin account
3. Add media libraries (`/media/library/movies`, `/media/library/tv`, etc.)
4. Install mobile apps

### Add to Home Assistant

1. **Settings → Devices & Services → "+ Add Integration"**
2. Search for **"Jellyfin"**
3. Enter URL: `http://<YOUR_ORANGE_PI_IP>:8096`
4. Enter your Jellyfin credentials

📖 Full details: [Part 3 - Jellyfin](./part3-media-voice.md)

---

## Section B: Local Voice Assistant ✅

All components run locally - **no cloud required!**

### Architecture

```
┌──────────────┐     ┌──────────────┐     ┌──────────────┐
│ Wake Word    │────▶│ Whisper      │────▶│ Home         │
│ (openWakeWord)│    │ (STT)        │     │ Assistant    │
└──────────────┘     └──────────────┘     └──────────────┘
         │                                       │
         │                ┌──────────────┐       │
         └────────────────│ Piper (TTS)  │◀──────┘
                          └──────────────┘
```

### Services Running

| Service | Port | Description |
|---------|------|-------------|
| Whisper | 10200 | Speech-to-text |
| Piper | 10201 | Text-to-speech |
| openWakeWord | 10400 | Wake word detection |

---

## Step 3.1: Install Wyoming Services ✅

### Create Directories

```bash
sudo mkdir -p /opt/wyoming/{whisper/data,piper/data,openwakeword/data}
sudo chown -R $USER:$USER /opt/wyoming
```

### Create docker-compose.yml

```bash
cat > /opt/wyoming/docker-compose.yml << "EOF"
services:
  # Speech-to-Text (Whisper)
  whisper:
    image: rhasspy/wyoming-whisper:latest
    container_name: whisper
    restart: unless-stopped
    ports:
      - "10200:10200"
    volumes:
      - ./whisper/data:/data
      - ./whisper/data:/cache
    environment:
      - TZ=Australia/Sydney
    command: --model small-int8 --language en

  # Text-to-Speech (Piper)
  piper:
    image: rhasspy/wyoming-piper:latest
    container_name: piper
    restart: unless-stopped
    ports:
      - "10201:10201"
    volumes:
      - ./piper/data:/data
    environment:
      - TZ=Australia/Sydney
    command: --voice en_US-lessac-low

  # Wake Word Detection
  openwakeword:
    image: rhasspy/wyoming-openwakeword:latest
    container_name: openwakeword
    restart: unless-stopped
    ports:
      - "10400:10400"
    volumes:
      - ./openwakeword/data:/data
    environment:
      - TZ=Australia/Sydney
EOF
```

### Start Services

```bash
cd /opt/wyoming && docker compose up -d
```

---

## Step 3.2: Configure Home Assistant Voice ⬜

### Add Wyoming Integrations

1. **Settings → Devices & Services**
2. Click **"+ Add Integration"**
3. Search for **"Wyoming"**
4. Add each service:

| Service | Host | Port |
|---------|------|------|
| Whisper | `<YOUR_ORANGE_PI_IP>` | 10200 |
| Piper | `<YOUR_ORANGE_PI_IP>` | 10201 |
| openWakeWord | `<YOUR_ORANGE_PI_IP>` | 10400 |

### Create Voice Pipeline

1. **Settings → Voice Assistants**
2. Click **"+ Add Assistant"**
3. Configure:
   - **Name:** "Local Assistant"
   - **Speech-to-Text:** Whisper
   - **Text-to-Speech:** Piper
   - **Wake Word:** hey_jarvis (or available options)
4. Save

### Test Voice Control

1. Click the **Assist** button in Home Assistant sidebar
2. Say "Hey Jarvis" (or your wake word)
3. Ask: "What time is it?"

---

## Available Wake Words

- `hey_jarvis`
- `alexa`
- `ok_nabu`

Custom wake words can be trained at: https://github.com/dscripka/openWakeWord

---

## Troubleshooting

### Whisper too slow on first run
- First transcription loads the model (~30 seconds)
- Subsequent transcriptions are faster
- Use `--model tiny-int8` for faster but less accurate results

### Piper voice not found
- Check available voices: https://github.com/rhasspy/piper/blob/master/VOICES.md
- Update command: `--voice en_US-lessac-low`

### Wake word not detecting
- Check microphone is working: `arecord -l`
- Test audio: `arecord -d 3 test.wav && aplay test.wav`
- Try speaking closer to the microphone

---

## Useful Commands

```bash
# Start Wyoming services
cd /opt/wyoming && docker compose up -d

# View logs
docker logs -f whisper
docker logs -f piper
docker logs -f openwakeword

# Restart a service
docker compose restart whisper

# Check ports
netstat -tlnp | grep -E "10200|10201|10400"
```

---

## Verification Checklist

**Voice Assistant:**
- [ ] Whisper running on port 10200
- [ ] Piper running on port 10201
- [ ] openWakeWord running on port 10400
- [ ] Wyoming integration added in Home Assistant
- [ ] Voice pipeline created
- [ ] Wake word detection working

---

## Progress Log

| Date | Step | Status |
|------|------|--------|
| 2026-02-27 22:27 | Jellyfin installed | ✅ Complete |
| 2026-02-27 22:38 | Jellyfin HA integration | ✅ Straightforward |
| 2026-02-27 23:03 | Wyoming services installed | ✅ Running |
| 2026-02-27 23:03 | Whisper (STT) running | ✅ Port 10200 |
| 2026-02-27 23:03 | Piper (TTS) running | ✅ Port 10201 |
| 2026-02-27 23:03 | openWakeWord running | ✅ Port 10400 |

---

## Next Steps

1. Add Wyoming integrations in Home Assistant
2. Create voice pipeline
3. Test wake word detection
4. Add microphone for voice input

---

*Part 3 - Media Server & Voice Assistant*  
*Last updated: 2026-02-27 23:03 GMT+11*