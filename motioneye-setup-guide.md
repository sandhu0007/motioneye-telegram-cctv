# MotionEye + Telegram + Tailscale CCTV Setup (Raspberry Pi)

This guide covers how to:
- Flash Raspberry Pi OS
- Install MotionEye for motion detection & recording
- Send motion snapshots to Telegram
- Access your CCTV remotely using Tailscale

---

## 🔧 1. Flash Raspberry Pi OS (Lite)

### Requirements
- Raspberry Pi Imager: https://www.raspberrypi.com/software/
- 8GB+ microSD card
- Raspberry Pi 4 or Zero W

### Steps
1. Insert SD card and open Raspberry Pi Imager.
2. Choose OS: **Raspberry Pi OS Lite (64-bit)**.
3. Click ⚙️ Settings (bottom right):
   - Enable SSH
   - Set hostname (e.g., `sandhucctv`)
   - Set Wi-Fi SSID & Password
4. Write the image and insert into Pi.
5. Boot the Pi and find IP using your router or `hostname -I`.

---

## 🎥 2. Install MotionEye

```bash
sudo apt update && sudo apt upgrade -y
sudo apt install -y python3-pip python3-dev curl libssl-dev libffi-dev libjpeg-dev zlib1g-dev libbz2-dev libx264-dev ffmpeg motion
pip install motioneye
sudo mkdir -p /etc/motioneye
sudo cp /usr/local/share/motioneye/extra/motioneye.conf.sample /etc/motioneye/motioneye.conf
sudo mkdir -p /var/lib/motioneye
sudo systemctl enable motioneye
sudo systemctl start motioneye
```

🟢 Access Web UI: `http://<RPI-IP>:8765`

---

## 📸 3. Add Camera in Web UI

1. Go to the MotionEye web UI.
2. Default user: `admin` (no password).
3. Add local V4L2 camera:
   - Device: `/dev/video0`
   - Set resolution: 640x480 or lower if needed.
4. Enable:
   - Still Images
   - Movies
   - Motion Detection

---

## 🤖 4. Send Photo to Telegram on Motion

### Telegram Setup
1. Message [@BotFather](https://t.me/BotFather) on Telegram.
2. Run `/newbot` and follow prompts.
3. Copy your Bot Token.
4. Get your chat ID: https://api.telegram.org/bot<token>/getUpdates

### Create Hook Script

```bash
sudo nano /etc/motioneye/telegram-snapshot.sh
```

Paste:

```bash
#!/bin/bash
TOKEN="your_bot_token"
CHAT_ID="your_chat_id"
FILE="$1"
curl -s -X POST "https://api.telegram.org/bot$TOKEN/sendPhoto" \
  -F chat_id="$CHAT_ID" \
  -F photo=@"$FILE" \
  -F caption="⚠️ Motion Detected"
```

```bash
sudo chmod +x /etc/motioneye/telegram-snapshot.sh
```

### Enable It in MotionEye
In MotionEye Web UI:
- Go to camera settings > Motion Notifications
- Set "Run a Command": `/etc/motioneye/telegram-snapshot.sh %f`

---

## 🌍 5. Access MotionEye via Tailscale (Optional)

```bash
curl -fsSL https://tailscale.com/install.sh | sh
sudo tailscale up
```

Copy your Pi’s Tailscale IP and open: `http://<tailscale-ip>:8765`

---

## ✅ Done!
You now have a Raspberry Pi-powered CCTV with Telegram alerts and remote access!
