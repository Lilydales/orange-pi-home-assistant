# Part 3: Media Server (Jellyfin)

**Time estimate:** 30-60 minutes  
**Difficulty:** Beginner  
**Status:** 🔄 In Progress - Jellyfin installed

---

## Overview

We'll install **Jellyfin** - a free, open-source media server with:
- Free hardware transcoding (no paywall)
- Live TV support
- Mobile apps
- No tracking or data collection

> **Why Jellyfin over Plex?**
> - Free hardware transcoding (Plex requires paid pass)
> - No paywalled features (skip intros, etc.)
> - Open source & privacy-focused
> - No account required

---

## Step 3.1: Prepare Directories ✅

**Executed on Orange Pi 5:**

```bash
# Create Jellyfin directories
sudo mkdir -p /opt/jellyfin/config
sudo mkdir -p /opt/jellyfin/cache
sudo chown -R $USER:$USER /opt/jellyfin

# Create media library directories
sudo mkdir -p /media/library/{movies,tv,music,photos}
sudo chown -R $USER:$USER /media/library
```

**Result:**
```
/opt/jellyfin/
├── config/
└── cache/

/media/library/
├── movies/
├── tv/
├── music/
└── photos/
```

---

## Step 3.2: Create Docker Compose File ✅

**Executed on Orange Pi 5:**

```bash
cat > /opt/jellyfin/docker-compose.yml << "EOF"
services:
  jellyfin:
    image: lscr.io/linuxserver/jellyfin:latest
    container_name: jellyfin
    restart: unless-stopped
    network_mode: host
    devices:
      - /dev/dri:/dev/dri
    volumes:
      - ./config:/config
      - ./cache:/cache
      - /media/library:/media:ro
    environment:
      - PUID=1000
      - PGID=1000
      - TZ=Australia/Sydney
      - JELLYFIN_PublishedServerUrl=<YOUR_ORANGE_PI_IP>
EOF
```

**File:** `/opt/jellyfin/docker-compose.yml`

---

## Step 3.3: Start Jellyfin ✅

**Executed on Orange Pi 5:**

```bash
cd /opt/jellyfin
docker compose up -d
```

**Result:**
```
CONTAINER ID   IMAGE                               STATUS
36ee9c6a7787   lscr.io/linuxserver/jellyfin:latest Up 5 seconds
```

---

## Step 3.4: Initial Setup ⬜

### Access Jellyfin

Open in browser: **http://<YOUR_ORANGE_PI_IP>:8096**

### Setup Wizard

1. **Language** - Select your language
2. **Create Account** - Set admin username and password
3. **Add Media Libraries**:
   - Click **"+ Add Media Library"**
   - Select content type (Movies, Shows, Music, etc.)
   - Browse to `/media/movies`, `/media/tv`, etc.
4. **Metadata Language** - English (or your preference)
5. **Remote Access** - Enable if you want access outside your network
6. **Finish**

---

## Step 3.5: Add Jellyfin to Home Assistant ⬜

Adding Jellyfin to Home Assistant is straightforward:

1. **Settings → Devices & Services**
2. Click **"+ Add Integration"**
3. Search for **"Jellyfin"**
4. Enter:
   - **URL:** `http://<YOUR_ORANGE_PI_IP>:8096`
   - **Username:** *(your Jellyfin admin username)*
   - **Password:** *(your Jellyfin admin password)*
5. Click **Submit**

This allows you to:
- Control playback from Home Assistant
- See media players as entities
- Create automations based on playback status

---

## Step 3.6: Add Media Libraries ⬜

In Jellyfin Dashboard → Libraries:

| Library Type | Folder Path | Notes |
|--------------|-------------|-------|
| Movies | `/media/movies` | Movie files |
| Shows | `/media/tv` | TV series |
| Music | `/media/music` | Music files |
| Photos | `/media/photos` | Photo albums |

### Supported Formats

**Video:** MP4, MKV, AVI, MOV, WMV  
**Audio:** MP3, FLAC, AAC, WAV, OGG  
**Images:** JPG, PNG, GIF, BMP

---

## Step 3.7: Hardware Transcoding (Optional) ⬜

If you have a GPU or hardware encoder:

1. Dashboard → Playback → transcoding
2. Enable **"Hardware acceleration"**
3. Select device: `VAAPI` or `VideoToolbox`

> **Note:** Orange Pi 5 has some GPU acceleration via V4L2/MPP. Results may vary.

---

## Step 3.8: Mobile Apps ⬜

Download Jellyfin app:
- **iOS:** App Store → "Jellyfin"
- **Android:** Play Store → "Jellyfin"
- **Fire TV:** Amazon Appstore → "Jellyfin"

Connect to: `http://<YOUR_ORANGE_PI_IP>:8096`

---

## Useful Commands

```bash
# Start Jellyfin
cd /opt/jellyfin && docker compose up -d

# View logs
docker logs -f jellyfin

# Restart
docker compose restart

# Check container
docker ps | grep jellyfin
```

---

## Verification Checklist

| Item | Status |
|------|--------|
| Jellyfin accessible at :8096 | ✅ |
| Setup wizard completed | ⬜ |
| Libraries created | ⬜ |
| Media files added | ⬜ |
| Hardware transcoding | ⬜ |
| Mobile app connected | ⬜ |

---

## Next Steps

1. Add your media files to `/media/library/`
2. Complete Jellyfin setup wizard
3. Install mobile apps
4. (Optional) Set up voice assistant - see Part 3B

---

## Progress Log

| Date | Step | Status |
|------|------|--------|
| 2026-02-27 22:27 | 3.1 - Create directories | ✅ Complete |
| 2026-02-27 22:27 | 3.2 - Create docker-compose.yml | ✅ Complete |
| 2026-02-27 22:27 | 3.3 - Pull Jellyfin image | ✅ Complete |
| 2026-02-27 22:27 | 3.4 - Start container | ✅ Running |
| 2026-02-27 22:38 | 3.5 - Add to Home Assistant | ✅ Straightforward via Integration |
| 2026-02-27 22:38 | 3.6 - Initial setup wizard | ⬜ Pending |
| 2026-02-27 22:38 | 3.7 - Add media libraries | ⬜ Pending |

---

*Part 3 - Media Server (Jellyfin)*  
*Last updated: 2026-02-27 22:38 GMT+11*