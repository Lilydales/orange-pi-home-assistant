# Part 0: Orange Pi 5 Initial Setup

**Before Home Assistant:** Get the hardware ready with WiFi/BT working

---

## Hardware

- Orange Pi 5 (8GB or 16GB)
- microSD card 16GB+ (Class 10 or better)
- AP6275P M.2 WiFi 6 + BT 5.0 module (official)
- Power supply (5V 4A USB-C recommended)
- Optional: M.2 NVMe SSD for storage

---

## 1. Download & Flash Image

### Image File

**Name:** `Orangepi5_1.2.2_ubuntu_jammy_server_linux6.1.99.7z`

**Download:** [Google Drive](https://drive.google.com/drive/folders/1i5zQOg1GIA4_VNGikFl2nPM0Y2MBw2M0)

### Flash Steps

1. Download and extract the `.7z` file with 7-Zip
2. Insert microSD card
3. Open **Raspberry Pi Imager**
4. Choose OS → Use custom → select the `.img` file
5. Choose Storage → select your microSD
6. Write → wait 5-15 min
7. Eject safely

*(Alternative: balenaEtcher)*

---

## 2. Install AP6275P WiFi/BT Module

### Physical Install

1. **Power off** Orange Pi 5 (unplug power)
2. Locate the **M.2 M-key slot** on the **bottom** of the board
3. Insert AP6275P module (align the notch)
4. Secure with the included screw
5. Insert microSD card (top slot)
6. Power on

### Enable WiFi in Software

**Default login:**
- User: `orangepi`
- Password: `orangepi`

```bash
sudo orangepi-config
```

**Menu path:**
- System → Hardware → `wifi-ap6275p`
- Press **Space** to enable
- Save/OK → Exit → Reboot

*This appends `overlays=wifi-ap6275p` to `/boot/orangepiEnv.txt`*

### Verify WiFi Works

```bash
# Check Broadcom chip detected
lspci | grep -i net

# Check wlan0 interface exists
ip link show wlan0

# Check kernel messages
dmesg | grep -i -E 'brcm|wifi|firm'
```

### Update Firmware

```bash
sudo apt update
sudo apt install --reinstall linux-firmware
sudo reboot
```

---

## 3. Connect to WiFi

### Using nmcli

```bash
# List available networks
nmcli device wifi list

# Connect to your network
nmcli device wifi connect "YourSSID" password "YourPassword"
```

### Using Text Menu

```bash
nmtui
```

### Verify Connection

```bash
ip addr show wlan0
ping 8.8.8.8
```

---

## 4. Enable SSH (Remote Access)

```bash
sudo apt install openssh-server
sudo systemctl enable ssh
sudo systemctl start ssh
sudo systemctl status ssh
```

### Find Your IP

```bash
ip addr show wlan0
# Look for "inet" line
```

### Connect from Another Machine

```bash
ssh orangepi@192.168.x.x
# Password: orangepi
```

**⚠️ Change password immediately:**

```bash
passwd
```

---

## 5. (Optional) Install M.2 NVMe SSD

If you have an NVMe drive for faster storage:

1. Power off
2. Insert NVMe in the M.2 slot (top side)
3. Boot from microSD
4. Format and mount:

```bash
# List drives
lsblk

# Format (adjust /dev/nvme0n1 as needed)
sudo mkfs.ext4 /dev/nvme0n1

# Create mount point
sudo mkdir -p /mnt/ssd

# Mount
sudo mount /dev/nvme0n1 /mnt/ssd

# Add to fstab for auto-mount
echo '/dev/nvme0n1 /mnt/ssd ext4 defaults 0 2' | sudo tee -a /etc/fstab
```

---

## 6. Update System

```bash
sudo apt update
sudo apt upgrade -y
sudo apt install -y curl wget git
```

---

## Post-Setup Checklist

- [ ] WiFi connected and working
- [ ] SSH enabled and accessible
- [ ] Password changed from default
- [ ] System updated
- [ ] (Optional) NVMe SSD mounted

---

## Next Step

→ **[Part 1: Home Assistant](./part1-home-assistant.md)**

---

## Quick Reference

| Task | Command |
|------|---------|
| Check WiFi hardware | `lspci \| grep -i net` |
| Check wlan0 interface | `ip link show wlan0` |
| List WiFi networks | `nmcli device wifi list` |
| Connect to WiFi | `nmcli device wifi connect "SSID" password "PASS"` |
| Text WiFi menu | `nmtui` |
| Check IP address | `ip addr show wlan0` |
| SSH status | `sudo systemctl status ssh` |
| Change password | `passwd` |