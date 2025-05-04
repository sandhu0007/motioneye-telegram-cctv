# 🧼 Raspberry Pi MotionEye + Telegram Bot: Clean Reinstall Guide

This guide will help you fully clean your Raspberry Pi and reinstall Raspberry Pi OS Lite, MotionEye, and the Telegram bot for motion alerts.

---

## 🔁 1. Reflash Raspberry Pi OS Lite (64-bit)

**Required: SD card reader + Raspberry Pi Imager**

1. Download [Raspberry Pi Imager](https://www.raspberrypi.com/software/)
2. Select:
   - OS: Raspberry Pi OS Lite (64-bit)
   - Storage: Your SD card
3. Optional (⚙️ icon):
   - Set hostname
   - Enable SSH
   - Set Wi-Fi credentials
   - Set username and password
4. Click **Write**. Then insert the SD card into your Pi and power on.

---

## 🔧 2. Setup MotionEye from scratch

```bash
sudo apt update && sudo apt upgrade -y

# Required packages
sudo apt install -y python3-venv python3-dev curl libssl-dev libffi-dev libjpeg-dev     zlib1g-dev libbz2-dev libx264-dev ffmpeg motion git

# Create a MotionEye virtual environment
python3 -m venv motioneye-venv
source motioneye-venv/bin/activate

# Install MotionEye
pip install motioneye

# Prepare configuration directories
sudo mkdir -p /etc/motioneye
sudo cp motioneye-venv/lib/python3.*/site-packages/motioneye/extra/motioneye.conf.sample /etc/motioneye/motioneye.conf
sudo mkdir -p /var/lib/motioneye
```

---

## 🔌 3. Create motionEye systemd service

```bash
sudo nano /etc/systemd/system/motioneye.service
```

Paste the following:

```ini
[Unit]
Description=MotionEye CCTV
After=network.target

[Service]
ExecStart=/home/pi/motioneye-venv/bin/python3 -m motioneye.meyectl startserver -c /etc/motioneye/motioneye.conf
WorkingDirectory=/home/pi
StandardOutput=inherit
StandardError=inherit
Restart=always

[Install]
WantedBy=multi-user.target
```

Then:

```bash
sudo systemctl daemon-reload
sudo systemctl enable motioneye
sudo systemctl start motioneye
```

---

## 📷 4. Add your camera

- Open browser and go to: `http://<your-pi-ip>:8765`
- Default user: **admin**
- Password: *(leave blank and set one)*

Add camera:
- Camera Type: Local V4L2
- Device: `/dev/video0` or `/dev/v4l/by-id/...`

---

## 🤖 5. Enable Telegram Bot Alert (Optional)

Follow the full setup from the GitHub repo:  
**https://github.com/sandhu0007/motioneye-telegram-cctv**

Make sure the bot script is placed correctly and `/etc/motioneye/camera-1.conf` uses:
```
on_picture_save /etc/motioneye/telegram-snapshot.sh
```

---

## 🧹 Cleaning Tip

To clean and reset:
```bash
sudo rm -rf /home/pi/*
sudo rm -rf /etc/motioneye /var/lib/motioneye
sudo rm -rf /usr/local/bin/telegram-watcher.sh /etc/systemd/system/telegram-watcher.service
sudo apt purge -y motion
sudo apt autoremove -y
sudo apt clean
```

Then reboot:
```bash
sudo reboot
```

---

© 2025 | Sandhu CCTV Bot Setup | Public domain license
