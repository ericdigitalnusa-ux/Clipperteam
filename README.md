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

### 1. Install Skill
```bash
# Create skill folder
mkdir -p ~/.hermes/skills/clipper/indigenous-australians-clipper/

# Copy SKILL.md to that folder
cp SKILL.md ~/.hermes/skills/clipper/indigenous-australians-clipper/
```

### 2. Setup Requirements

#### FFmpeg
```bash
# Ubuntu/Debian
sudo apt install ffmpeg

# Check installation
ffmpeg -version
```

#### Python Dependencies
```bash
pip install faster-whisper
```

#### Poppins Font
```bash
# Download font
wget -O Poppins.zip "https://fonts.google.com/download?family=Poppins"

# Extract and install
unzip -o Poppins.zip -d ~/.fonts/
fc-cache -fv
```

---

## 📱 Repliz Account Setup

### Get Repliz API Key
1. Login to https://repliz.com
2. Go to Settings → API Keys
3. Copy your API key

### Setup Environment
```bash
export REPLIZ_API_KEY="your_api_key_here"
```

### Repliz Account ID
Each agent needs a Repliz account ID for scheduling:
- **Default Account:** `6a8a365462caae1e0402a583`

### Schedule Video to Repliz
```bash
AUTH=$(echo -n "REPLIZ_API_KEY" | base64)
ACCOUNT_ID="6a8a365462caae1e0402a583"
VIDEO_URL="https://your-domain.com/video.mp4"

curl -X POST "https://api.repliz.com/public/schedule" \
  -H "Authorization: Basic ${AUTH}" \
  -H "Content-Type: application/json" \
  -d '{
    "accountId": "'${ACCOUNT_ID}'",
    "type": "video",
    "title": "Your Video Title",
    "description": "Video description...",
    "medias": [{"type": "video", "url": "'${VIDEO_URL}'"}],
    "scheduleAt": "2026-09-01T14:00:00Z"
  }'
```

---

## 🎬 Clipper Workflow

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
python3 indigenous_australians_clip.py
```

### Step 4: Upload to VPS
```bash
# Upload video
scp clips/MwLuPhwpViI/processed/v26/final.mp4 user@vps:/var/www/html/videos/

# Or copy to public folder
cp clips/MwLuPhwpViI/processed/v26/final.mp4 /var/www/html/videos/
```

### Step 5: Schedule to Repliz
```bash
# Schedule for posting
curl -X POST "https://api.repliz.com/public/schedule" \
  -H "Authorization: Basic ${AUTH}" \
  -H "Content-Type: application/json" \
  -d '{
    "accountId": "'${ACCOUNT_ID}'",
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
│               ├── clip.ass      # Subtitle file
│               └── final.mp4     # Output video
├── indigenous_australians_clip.py
├── SKILL.md
└── README.md
```

---

## ⚙️ Subtitle Style Settings

### Subtitle (Bottom)
```
Style: Sub,Poppins Bold,65,&H00FFFFFF,&H000000FF,&H00000000,&HCC000000,-1,0,0,0,100,100,0,0,1,3,1.5,2,15,15,100,1
```
- Font: Poppins Bold
- Size: 65px
- Color: White
- Position: Bottom letterbox (MarginV=100)
- Background: Semi-transparent black

### Source Credit (Top)
```
Style: Src,Poppins Bold,36,&H00FFFF00,&H000000FF,&H00000000,&H00000000,-1,0,0,0,100,100,0,0,1,2.5,2,8,10,10,20,1
```
- Font: Poppins Bold
- Size: 36px
- Color: Yellow (#FFFF00)
- Position: Top center

---

## 🔧 FFmpeg Filter Explained

### Cinematic Blur Background
```bash
# 1. Scale to 1280x720 (standardize)
scale=1280:720:flags=lanczos,setsar=1[base]

# 2. Split for background and foreground
[base]split=2[bga][fga]

# 3. Background: scale up, crop to 9:16, apply blur
[bga]scale=720:1280:force_original_aspect_ratio=increase,crop=720:1280,boxblur=15:2[bg]

# 4. Foreground: scale down to fit 9:16 (adds black bars)
[fga]scale=720:1280:force_original_aspect_ratio=decrease[fg]

# 5. Overlay foreground centered on blurred background
[bg][fg]overlay=(W-w)/2:(H-h)/2[vid]
```

---

## 📋 YouTube Channels to Clip

| Channel | URL | Topic |
|---------|-----|-------|
| ABC Australia | https://youtu.be/MwLuPhwpViI | Documentary |
| Akbar Faizal | https://www.youtube.com/@AkbarFaizalUncensored | Commentary |
| (Add more) | | |

---

## 🔐 Environment Variables

```bash
# Required
export REPLIZ_API_KEY="your_repliz_api_key"

# Optional
export VPS_USER="your_ssh_user"
export VPS_HOST="your-vps.com"
export VPS_PATH="/var/www/html/videos/"
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

### Repliz schedule failed?
```bash
# Check API key
curl -H "Authorization: Basic $(echo -n $REPLIZ_API_KEY | base64)" \
  https://api.repliz.com/public/accounts
```

---

## 📝 License
Free to use for Hermes Agent workflows.
