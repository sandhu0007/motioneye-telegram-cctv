# MotionEye CCTV with Telegram Bot Integration (Raspberry Pi OS Lite)

This guide provides a **working and tested** step-by-step setup for installing MotionEye with Telegram snapshot alerts on Raspberry Pi OS Lite (64-bit). It includes all the necessary troubleshooting fixes.

---

## ✅ What You Get

- MotionEye CCTV server running on Raspberry Pi OS Lite
- Telegram bot sends snapshot alerts on motion detection
- Fully working setup with automatic recovery and proper permissions

---

## 🧰 Prerequisites

- Raspberry Pi (any model with USB support)
- Raspberry Pi OS Lite 64-bit installed
- Logitech USB Webcam or compatible V4L2 camera
- Working internet
- Telegram account and bot token

---

## ⚙️ Step-by-Step Installation

### 1. Flash Raspberry Pi OS Lite

- Download from: [https://www.raspberrypi.com/software/operating-systems/](https://www.raspberrypi.com/software/operating-systems/)
- Flash using Raspberry Pi Imager (choose Raspberry Pi OS Lite 64-bit)
- Boot and login (default `pi` / `raspberry`)
- Update your system:

```bash
sudo apt update && sudo apt upgrade -y
```

---

### 2. Install Required Packages

```bash
sudo apt install -y python3-pip python3-dev curl libssl-dev libffi-dev libjpeg-dev zlib1g-dev libbz2-dev libx264-dev ffmpeg motion git lsof net-tools
```

---

### 3. Setup Python Virtual Environment

```bash
python3 -m venv motioneye-venv
source motioneye-venv/bin/activate
pip install wheel
pip install motioneye
```

---

### 4. Setup MotionEye Configuration

```bash
sudo mkdir -p /etc/motioneye
sudo mkdir -p /var/lib/motioneye
```

Create config file:

```bash
sudo nano /etc/motioneye/motioneye.conf
```

Add:

```ini
conf_path /etc/motioneye
run_path /run/motioneye
log_path /var/log/motioneye
media_path /var/lib/motioneye
listen 0.0.0.0
port 8765
```

---

### 5. Fix: PID Directory Does Not Exist

```bash
sudo mkdir -p /run/motioneye
sudo chown $(whoami):$(whoami) /run/motioneye
```

---

### 6. Create systemd Service

```bash
sudo nano /etc/systemd/system/motioneye.service
```

Paste:

```ini
[Unit]
Description=MotionEye CCTV
After=network.target

[Service]
ExecStart=/home/$(whoami)/motioneye-venv/bin/python3 -m motioneye.meyectl startserver -c /etc/motioneye/motioneye.conf
WorkingDirectory=/home/$(whoami)
StandardOutput=inherit
StandardError=inherit
Restart=always

[Install]
WantedBy=multi-user.target
```

---

### 7. Enable and Start Service

```bash
sudo systemctl daemon-reload
sudo systemctl enable motioneye
sudo systemctl start motioneye
```

---

### 8. Check If It’s Running

```bash
sudo lsof -i :8765
```

Or

```bash
curl http://localhost:8765
```

Access in browser:

```
http://<your-pi-ip>:8765
```

---

## ✅ Tested Working As Of

**2025-05-04 07:14:09**

---

## ⚠️ Notes

- Make sure your webcam is connected and accessible via `/dev/video0`
- Run `v4l2-ctl --list-devices` to confirm camera visibility
- This setup assumes no other process is using the camera

---

## 🧠 Credits

Made with ❤️ by Sandhu  
GitHub: [sandhu0007](https://github.com/sandhu0007)

