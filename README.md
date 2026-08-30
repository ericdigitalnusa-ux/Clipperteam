# 🎬 Clipperteam

Hermes Agent video clipping pipelines. Automated YouTube → TikTok/Shorts workflow.

---

## 📺 Available Skills

### 1. Indigenous Australians Clipper
| Property | Value |
|----------|-------|
| **Source** | https://youtu.be/MwLuPhwpViI |
| **Channel** | ABC Australia - "You Can't Ask That" |
| **Topic** | Indigenous Australians documentary |
| **Output** | 9:16 vertical 720×1280 |
| **Subtitle** | Poppins Bold 65px, white, bottom letterbox |
| **Source Credit** | Yellow, top center |
| **Background** | Cinematic blur |
| **Duration** | 59 seconds |

### 2. Akbar Faizal Clipper *(Coming Soon)*
| Property | Value |
|----------|-------|
| **Channel** | https://www.youtube.com/@AkbarFaizalUncensored |
| **Topic** | Indonesian commentary |
| **Output** | 9:16 vertical 720×1280 |

---

## 🚀 Quick Start

### 1. Clone & Install
```bash
git clone https://github.com/ericdigitalnusa-ux/Clipperteam.git /tmp/Clipperteam
cp -r /tmp/Clipperteam/skills/* ~/.hermes/skills/
cp /tmp/Clipperteam/skills/indigenous-australians-clipper/*.py ~/clipper-company/
```

### 2. Setup Requirements

#### FFmpeg
```bash
sudo apt install ffmpeg
```

#### Python Dependencies
```bash
pip install faster-whisper
```

#### Poppins Font
```bash
wget -O Poppins.zip "https://fonts.google.com/download?family=Poppins"
unzip -o Poppins.zip -d ~/.fonts/
fc-cache -fv
```

---

## 🔐 Environment Variables

Each agent needs their own credentials:

```bash
# VPS Upload (where to store videos)
export VPS_HOST="your-vps.com"
export VPS_USER="your_ssh_username"
export VPS_KEY="/path/to/your/ssh_key"

# Repliz Account (where to schedule posts)
export REPLIZ_API_KEY="your_repliz_api_key"
export REPLIZ_ACCOUNT_ID="your_repliz_account_id"
```

### How to get credentials:

**VPS:**
- Host: Your VPS domain/IP (e.g., `vps.mydomain.com`)
- User: SSH username
- Key: Generate SSH key pair, add public key to VPS

**Repliz:**
1. Login to https://repliz.com
2. Settings → API Keys → Create new
3. Account ID: Found in account settings

---

## ⏰ Cronjob Setup (Auto Clip Daily)

### Server Architecture
```
┌─────────────────┐     ┌─────────────────┐     ┌─────────────────┐
│  Agent Server   │────▶│   VPS Server    │────▶│     Repliz      │
│  (Download)     │ SCP │   (Storage)     │ API │   (Schedule)    │
└─────────────────┘     └─────────────────┘     └─────────────────┘
```

Each agent has:
- ✅ Own agent server (download video)
- ✅ Own VPS (store & serve video)
- ✅ Own Repliz account (schedule posts)

### Create Cronjob

```bash
hermes cron create \
  --name "Indigenous Clipper" \
  --schedule "0 7 * * *" \
  --skills indigenous-australians-clipper \
  --prompt "Download Indigenous Australians video to agent server, SCP upload to VPS (host: \$VPS_HOST, user: \$VPS_USER, key: \$VPS_KEY, path: /var/www/html/videos/), schedule to Repliz (api_key: \$REPLIZ_API_KEY, account_id: \$REPLIZ_ACCOUNT_ID)" \
  --env VPS_HOST,VPS_USER,VPS_KEY,REPLIZ_API_KEY,REPLIZ_ACCOUNT_ID \
  --deliver origin
```

### Manual Cron Setup (Alternative)
```bash
# 1. Create cron via Hermes CLI
/hermes/path/cron create

# 2. Or via API
curl -X POST "http://localhost:PORT/api/cron" \
  -H "Authorization: Bearer TOKEN" \
  -d '{
    "name": "Indigenous Clipper",
    "schedule": "0 7 * * *",
    "skills": ["indigenous-australians-clipper"],
    "prompt": "Download video, upload to VPS, schedule to Repliz",
    "env": ["VPS_HOST", "VPS_USER", "VPS_KEY", "REPLIZ_API_KEY", "REPLIZ_ACCOUNT_ID"],
    "deliver": "origin"
  }'
```

### Cron Workflow
```
1. Trigger: Daily at 07:00 UTC
2. Download: yt-dlp video to agent server
3. Transcribe: faster-whisper language detection
4. Generate: 59s clip with Gen-Z subtitles
5. Upload: SCP to VPS /var/www/html/videos/
6. Schedule: Repliz API post
7. Notify: Send result to origin chat
```

---

## 🎬 Manual Workflow

### Step 1: Download Video
```bash
VIDEO_ID=MwLuPhwpViI
yt-dlp -f 18 \
  -o "downloads/${VIDEO_ID}_full.%(ext)s" \
  "https://youtube.com/watch?v=${VIDEO_ID}"
```

### Step 2: Transcribe
```bash
ffmpeg -y -i "downloads/${VIDEO_ID}_full.mp4" \
  -vn -acodec pcm_s16le -ar 16000 -ac 1 \
  /tmp/${VIDEO_ID}_audio.wav

python3.12 -c "
from faster_whisper import WhisperModel
m = WhisperModel('base', device='cpu', compute_type='int8')
segs, _ = m.transcribe('/tmp/${VIDEO_ID}_audio.wav', language='en', vad_filter=True)
import json
print(json.dumps([{'start': s.start, 'end': s.end, 'text': s.text.strip()} for s in segs]))
" > /tmp/transcript_full.json
```

### Step 3: Generate Clip
```bash
cd ~/clipper-company
python3 indigenous_australians_clip.py
```

### Step 4: Upload to VPS
```bash
scp -i $VPS_KEY clips/MwLuPhwpViI/processed/v26/final.mp4 \
  $VPS_USER@$VPS_HOST:/var/www/html/videos/
```

### Step 5: Schedule to Repliz
```bash
AUTH=$(echo -n "$REPLIZ_API_KEY" | base64)

curl -X POST "https://api.repliz.com/public/schedule" \
  -H "Authorization: Basic ${AUTH}" \
  -H "Content-Type: application/json" \
  -d '{
    "accountId": "'${REPLIZ_ACCOUNT_ID}'",
    "type": "video",
    "title": "What the government gives us...",
    "topic": "Entertainment",
    "medias": [{"type": "video", "url": "https://your-domain.com/videos/video.mp4"}],
    "scheduleAt": "2026-09-01T14:00:00Z"
  }'
```

---

## 📁 Folder Structure
```
clipper-company/
├── downloads/
│   └── MwLuPhwpViI_full.mp4
├── clips/
│   └── MwLuPhwpViI/
│       └── processed/
│           └── v26/
│               ├── clip.ass
│               └── final.mp4
├── indigenous_australians_clip.py
└── SKILL.md

~/.hermes/skills/
└── indigenous-australians-clipper/
    ├── SKILL.md
    └── indigenous_australians_clip.py
```

---

## ⚙️ Subtitle Settings

| Element | Font | Size | Color | Position |
|---------|------|------|-------|----------|
| Subtitle | Poppins Bold | 65px | White | Bottom letterbox (MarginV=100) |
| Source | Poppins Bold | 36px | Yellow | Top center |
| Hook | Poppins Bold | 40px | White | Center with shadow |
| CTA | Poppins SemiBold | 20px | White | Bottom |

### ASS Style Strings
```
Subtitle: Style: Sub,Poppins Bold,65,&H00FFFFFF,&H000000FF,&H00000000,&HCC000000,-1,0,0,0,100,100,0,0,1,3,1.5,2,15,15,100,1
Source:   Style: Src,Poppins Bold,36,&H00FFFF00,&H000000FF,&H00000000,&H00000000,-1,0,0,0,100,100,0,0,1,2.5,2,8,10,10,20,1
```

---

## 🔧 FFmpeg Filter

```bash
# Cinematic Blur Background
-vf "scale=1280:720:flags=lanczos,setsar=1[base];
[base]split=2[bga][fga];
[bga]scale=720:1280:force_original_aspect_ratio=increase,crop=720:1280,boxblur=15:2[bg];
[fga]scale=720:1280:force_original_aspect_ratio=decrease[fg];
[bg][fg]overlay=(W-w)/2:(H-h)/2[vid]"
```

---

## 🐛 Troubleshooting

### Video not downloading?
```bash
yt-dlp --verbose "https://youtube.com/watch?v=VIDEO_ID"
```

### Whisper not working?
```bash
pip install --upgrade faster-whisper
python3 -c "from faster_whisper import WhisperModel; print('OK')"
```

### Font not showing?
```bash
fc-cache -fv
fc-list | grep Poppins
```

### VPS upload failed?
```bash
# Test SSH connection
ssh -i $VPS_KEY $VPS_USER@$VPS_HOST "echo OK"
```

### Repliz schedule failed?
```bash
# Verify credentials
curl -H "Authorization: Basic $(echo -n $REPLIZ_API_KEY | base64)" \
  https://api.repliz.com/public/accounts
```

---

## 📋 YouTube Channels

| Channel | URL | Topic |
|---------|-----|-------|
| ABC Australia | https://youtu.be/MwLuPhwpViI | Documentary |
| Akbar Faizal | https://www.youtube.com/@AkbarFaizalUncensored | Commentary |
| (Add more) | | |

---

## 📝 License
Free to use for Hermes Agent workflows.
