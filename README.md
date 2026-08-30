# 🎬 Clipperteam

Hermes Agent video clipping pipelines. Automated YouTube → TikTok/Shorts workflow.

---

## ⚠️ SYSTEM REQUIREMENTS

**WAJIB di-install sebelum run skill ini:**

### 1. FFmpeg
```bash
# Ubuntu/Debian
sudo apt update && sudo apt install ffmpeg

# CentOS/RHEL
sudo yum install ffmpeg

# macOS
brew install ffmpeg

# Verify
ffmpeg -version
```

### 2. yt-dlp
```bash
pip install yt-dlp
# atau
pip3 install yt-dlp
```

### 3. faster-whisper (Speech-to-Text)
```bash
pip install faster-whisper
# atau
pip3 install faster-whisper
```

### 4. Poppins Font
```bash
# Download
wget -O /tmp/Poppins.zip "https://fonts.google.com/download?family=Poppins"

# Extract & install
sudo unzip -o /tmp/Poppins.zip -d /usr/share/fonts/truetype/
mkdir -p ~/.fonts
unzip -o /tmp/Poppins.zip -d ~/.fonts/

# Refresh font cache
fc-cache -fv

# Verify
fc-list | grep -i poppins
```

### 5. Python Environment
```bash
# Pastikan Python 3.8+
python3 --version

# Virtual environment (recommended)
python3 -m venv ~/clipper-venv
source ~/clipper-venv/bin/activate
pip install faster-whisper yt-dlp
```

---

## 🚀 QUICK START

### 1. Clone & Install Skill
```bash
git clone https://github.com/ericdigitalnusa-ux/Clipperteam.git /tmp/Clipperteam
cp -r /tmp/Clipperteam/skills/* ~/.hermes/skills/
```

### 2. Setup Folder Structure
```bash
mkdir -p ~/clipper-company/{downloads,clips}
cd ~/clipper-company
```

### 3. Setup Environment Variables
```bash
# VPS (untuk simpan video)
export VPS_HOST="your-vps.com"
export VPS_USER="your_ssh_username"
export VPS_KEY="/path/to/your/ssh_key"

# Repliz (untuk schedule posting)
export REPLIZ_API_KEY="your_repliz_api_key"
export REPLIZ_ACCOUNT_ID="your_repliz_account_id"
```

### 4. Run Skill
```bash
# Load skill di Hermes
# Skill akan handle:
# - Download video
# - Transcribe
# - Generate clip
# - Upload ke VPS
# - Schedule ke Repliz
```

---

## 📁 Folder Structure
```
~/clipper-company/
├── downloads/           # Video original
│   └── *_full.mp4
├── clips/              # Output clip
│   └── */processed/
│       └── v*/final.mp4
├── indigenous_australians_clip.py
└── ishowspeed_clip.py
```

---

## 🔧 Troubleshooting

### "ffmpeg: command not found"
```bash
sudo apt install ffmpeg
```

### "No module named 'faster_whisper'"
```bash
pip install faster-whisper
# atau dengan venv
source ~/clipper-venv/bin/activate
pip install faster-whisper
```

### "yt-dlp: command not found"
```bash
pip install yt-dlp
```

### "Font not found: Poppins"
```bash
# Install ulang font
fc-cache -fv
fc-list | grep -i poppins
```

### "Permission denied" (VPS upload)
```bash
# Test SSH connection
ssh -i $VPS_KEY $VPS_USER@$VPS_HOST "echo OK"

# Pastikan SSH key punya permission benar
chmod 600 $VPS_KEY
```

### "Repliz API error"
```bash
# Verify API key
curl -H "Authorization: Basic $(echo -n $REPLIZ_API_KEY | base64)" \
  https://api.repliz.com/public/accounts
```

---

## 📺 Available Skills

### 1. Indigenous Australians Clipper
- **Source:** https://youtu.be/MwLuPhwpViI
- **Channel:** ABC Australia
- **Output:** 9:16 vertical 720×1280

### 2. iShowSpeed Clipper
- **Channel:** https://www.youtube.com/@IShowSpeed
- **Content:** Speed Goes Pro series
- **Output:** 9:16 vertical 720×1280

---

## ⏰ Cronjob Setup

```bash
hermes cron create \
  --name "Clipper Daily" \
  --schedule "0 7 * * *" \
  --skills indigenous-australians-clipper \
  --prompt "Download video, generate clip, upload, schedule to Repliz" \
  --deliver origin
```

---

## 📝 License
Free to use for Hermes Agent workflows.
