# Part 2: Frigate NVR with Local Object Detection

**Time estimate:** 30-60 minutes  
**Difficulty:** Intermediate  
**Status:** ✅ Complete

---

## Overview

Frigate is an NVR (Network Video Recorder) with real-time AI object detection. It detects people, cars, pets, and more - all processed locally on your Orange Pi 5.

---

## Step 2.1: Prepare Directories ✅

**Executed on Orange Pi 5:**

```bash
# Create Frigate directories
sudo mkdir -p /opt/frigate/config
sudo mkdir -p /opt/frigate/media/clips
sudo mkdir -p /opt/frigate/media/recordings
sudo chown -R $USER:$USER /opt/frigate
```

**Result:**
```
/opt/frigate/
├── config/
│   └── config.yml
└── media/
    ├── clips/
    └── recordings/
```

---

## Step 2.2: Create Frigate Configuration ✅

**Executed on Orange Pi 5:**

```bash
cat > /opt/frigate/config/config.yml << "EOF"
# Frigate Configuration for Orange Pi 5
# Add your cameras under the cameras: section

mqtt:
  enabled: false  # Enable when you install Mosquitto

# Detectors - using CPU for now (add Coral/Hailo later)
detectors:
  cpu:
    type: cpu
    num_threads: 3

# Model configuration
model:
  path: /cpu_model.tflite

# Objects to detect
objects:
  track:
    - person
    - car
    - dog
    - cat
  filters:
    person:
      min_score: 0.5
      threshold: 0.7

# Camera placeholder - add your cameras here
# Example:
# cameras:
#   front_door:
#     ffmpeg:
#       inputs:
#         - path: rtsp://username:password@camera-ip:554/stream1
#           roles:
#             - detect
#             - record
#     detect:
#       width: 1920
#       height: 1080
#       fps: 5
#     record:
#       enabled: true
#       retain:
#         days: 7

# UI Settings  
ui:
  live_mode: jsmpeg
EOF
```

**File:** `/opt/frigate/config/config.yml`

---

## Step 2.3: Docker Compose for Frigate ✅

**Executed on Orange Pi 5:**

```bash
cat > /opt/frigate/docker-compose.yml << "EOF"
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
      - "8971:8971"   # Web UI
      - "8554:8554"   # RTSP
      - "8555:8555"   # WebRTC
      - "8555:8555/udp"
    environment:
      - TZ=Australia/Sydney
EOF
```

**File:** `/opt/frigate/docker-compose.yml`

---

## Step 2.4: Start Frigate ✅

**Executed on Orange Pi 5:**

```bash
cd /opt/frigate

# Pull and start (image is ~900MB)
docker compose pull
docker compose up -d

# Check logs
docker logs -f frigate
```

**Result:**
```
CONTAINER ID   IMAGE                                    STATUS
0e6c68a164a0   ghcr.io/blakeblackshear/frigate:stable   Up (healthy)
```

---

## Step 2.6: Add Cameras ✅

### Mosquitto MQTT Broker (Optional)

> **Note:** MQTT is optional. Frigate cameras work via API. MQTT is only needed if you want detection events in Home Assistant for automations.

**Location:** `/opt/mosquitto/`  
**Port:** 1883

```bash
# Install Mosquitto (optional - skip if you don't need MQTT events)
sudo mkdir -p /opt/mosquitto/{config,data,log}
sudo chown -R $USER:$USER /opt/mosquitto

cat > /opt/mosquitto/config/mosquitto.conf << "EOF"
listener 1883
allow_anonymous true
persistence true
persistence_location /mosquitto/data
EOF

cat > /opt/mosquitto/docker-compose.yml << "EOF"
services:
  mosquitto:
    image: eclipse-mosquitto:2
    container_name: mosquitto
    restart: unless-stopped
    ports:
      - "1883:1883"
    volumes:
      - ./config:/mosquitto/config
      - ./data:/mosquitto/data
EOF

cd /opt/mosquitto && docker compose up -d
```

### Camera Configuration

Edit `/opt/frigate/config/config.yml`:

```yaml
# Frigate Configuration for Orange Pi 5

mqtt:
  host: <YOUR_ORANGE_PI_IP>
  port: 1883

detectors:
  cpu:
    type: cpu
    num_threads: 3

model:
  path: /cpu_model.tflite

objects:
  track:
    - person
  filters:
    person:
      min_score: 0.6
      threshold: 0.7

go2rtc:
  streams:
    living_room_camera:
      - rtsp://<YOUR_CAMERA_IP>:8554/living-room-camera?video=all&audio=all

cameras:
  living_room_camera:
    ffmpeg:
      inputs:
        - path: rtsp://<YOUR_CAMERA_IP>:8554/living-room-camera?video=all&audio=all
          roles:
            - detect
            - record
            - audio
      output_args:
        record: preset-record-generic-audio-copy
    detect:
      width: 1280
      height: 720
      fps: 5
    audio:
      enabled: true
      listen:
        - speech
    record:
      enabled: true
      retain:
        days: 2
    snapshots:
      enabled: true
      retain:
        default: 2
    birdseye:
      mode: objects

ui:
  live_mode: jsmpeg
```

After editing, restart Frigate:

```bash
cd /opt/frigate && docker compose restart
```

---

## Step 2.7: Access Frigate UI ✅

Open in browser: **https://<YOUR_ORANGE_PI_IP>:8971** (accept certificate warning)

**Login credentials** (auto-generated on first startup):
- **Username:** `admin`
- **Password:** Run `docker logs frigate 2>&1 | grep -A1 "User:"` to view

> **Tip:** Change the password in Frigate Settings after first login.

---

## Step 2.7: Add Frigate Integration to Home Assistant ⬜

After HACS is configured in Home Assistant, add the Frigate integration:

### Install via HACS

1. In Home Assistant, go to **HACS → Integrations**
2. Click **"+ Explore & Download Repositories"**
3. Search for **"Frigate"**
4. Click **"Download"**
5. Restart Home Assistant:
   ```bash
   cd /opt/homeassistant && docker compose restart
   ```

### Configure the Integration

1. Go to **Settings → Devices & Services**
2. Click **"+ Add Integration"**
3. Search for **"Frigate"**
4. Enter the following:
   - **URL:** `https://<YOUR_ORANGE_PI_IP>:8971` (use **https://**, accept certificate warning)
   - **Username:** `admin`
   - **Password:** *(your Frigate password - see below)*

> **⚠️ Important:** This is the **Docker version** of Frigate, not the Home Assistant add-on. You must enter the IP address and login credentials manually.

### Enable Camera in Home Assistant

After adding the integration, you must enable WebRTC for cameras to appear:

1. Go to **Settings → Devices & Services → Frigate**
2. Click **Configure** (gear icon)
3. ✅ Tick **"Use Frigate-native WebRTC support"**
4. Click **Submit**

The camera should now appear in Home Assistant!

### Frigate Login Credentials

When Frigate starts for the first time, it auto-generates a random password shown in the logs:

```bash
# View the generated password
docker logs frigate 2>&1 | grep -A1 "User:"
```

**Example output:**
```
INFO: ***    User: admin                                   ***
INFO: ***    Password: 97cbb3bb654f2a6110083d74bb87c1fd   ***
```

> **Tip:** Change this password in Frigate's Settings after your first login.

**📖 Full integration guide:** https://github.com/blakeblackshear/frigate-hass-integration

---

## Step 2.8: Hardware Acceleration (Optional)

> **This guide uses CPU detection.** Coral TPU or Hailo-8 can be added later for faster detection, but are not required.

| Option | Detection Speed | Cost |
|--------|-----------------|------|
| **CPU Only** (current) | Moderate | $0 |
| Coral TPU (USB) | Fast | ~$100 AUD |
| Hailo-8 (M.2) | Very fast | ~$150 AUD |

---

## Useful Commands

```bash
# View Frigate logs
docker logs -f frigate

# Restart Frigate
cd /opt/frigate && docker compose restart

# Check camera streams
ffprobe rtsp://username:password@camera-ip:554/stream1

# View recordings
ls -la /opt/frigate/media/recordings/
```

---

## Verification Checklist

| Item | Status |
|------|--------|
| Frigate UI accessible | ✅ |
| Config file created | ✅ |
| Container healthy | ✅ |
| Cameras configured | ⬜ |
| Home Assistant integration | ⬜ |

---

## Troubleshooting

### Camera not connecting
```bash
# Test RTSP stream
ffprobe rtsp://username:password@camera-ip:554/stream1
```

### High CPU usage
- Lower detection FPS
- Use smaller resolution for detection
- Add Coral/Hailo accelerator

### No detections
- Check `min_score` settings
- Verify camera timestamp is correct
- Check logs: `docker logs frigate | grep -i detect`

---

## Next Steps

1. Add your cameras to `/opt/frigate/config/config.yml`
2. Restart Frigate: `docker compose restart`
3. Install Mosquitto MQTT broker for notifications (optional)
4. Continue to [Part 3: Media Server & Voice Assistant](./part3-media-voice.md)

---

## Progress Log

| Date | Step | Status |
|------|------|--------|
| 2026-02-27 21:26 | 2.1 - Create directories | ✅ Complete |
| 2026-02-27 21:26 | 2.2 - Create config.yml | ✅ Complete |
| 2026-02-27 21:26 | 2.3 - Create docker-compose.yml | ✅ Complete |
| 2026-02-27 21:32 | 2.4 - Pull Frigate image | ✅ Complete |
| 2026-02-27 21:35 | 2.5 - Start container | ✅ Complete |
| 2026-02-27 21:46 | 2.5 - Access via HTTPS | ✅ Complete |
| 2026-02-27 21:52 | 2.5 - Login (auto-generated password) | ✅ Complete |
| 2026-02-27 22:06 | 2.6 - Install Mosquitto MQTT | ✅ Complete |
| 2026-02-27 22:07 | 2.7 - Add living room camera | ✅ Complete |
| 2026-02-27 22:23 | 2.10 - WebRTC tip for HA | 📝 Important! |

---

*Part 2 - Frigate NVR*  
*Last updated: 2026-02-27 22:23 GMT+11*