# 🎬 Clipperteam

Hermes Agent video clipping pipelines. Automated YouTube → TikTok/Shorts workflow.

## Available Skills

### 📺 Indigenous Australians Clipper
Gen-Z style documentary clips from ABC Australia - "You Can't Ask That"

| Property | Value |
|----------|-------|
| **Source** | https://youtu.be/MwLuPhwpViI |
| **Output** | 9:16 vertical 720×1280 |
| **Subtitle** | Poppins Bold 65px, white, bottom letterbox |
| **Source Credit** | Yellow, top center |
| **Background** | Cinematic blur (Akbar Faizal style) |
| **Duration** | 59 seconds |

**Subtitle Settings:**
```
Style: Sub,Poppins Bold,65,&H00FFFFFF,&H000000FF,&H00000000,&HCC000000,-1,0,0,0,100,100,0,0,1,3,1.5,2,15,15,100,1
```

**Source Credit Settings:**
```
Style: Src,Poppins Bold,36,&H00FFFF00,&H000000FF,&H00000000,&H00000000,-1,0,0,0,100,100,0,0,1,2.5,2,8,10,10,20,1
```

---

## 🚀 Quick Start

### 1. Install Skill
Copy `SKILL.md` file to your Hermes skills folder:
```bash
~/.hermes/skills/clipper/indigenous-australians-clipper/SKILL.md
```

### 2. Download Video
```bash
VIDEO_ID=MwLuPhwpViI
yt-dlp -f 18 -o "downloads/${VIDEO_ID}_full.%(ext)s" \
  "https://youtube.com/watch?v=${VIDEO_ID}"
```

### 3. Transcribe
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

### 4. Run Clip Script
```bash
python3 indigenous_australians_clip.py
```

### 5. Output
Video saved to: `clips/MwLuPhwpViI/processed/v26/final.mp4`

---

## 🎨 FFmpeg Filter (Cinematic Blur)

```bash
-vf "scale=1280:720:flags=lanczos,setsar=1[base];
[base]split=2[bga][fga];
[bga]scale=720:1280:force_original_aspect_ratio=increase,crop=720:1280,boxblur=15:2[bg];
[fga]scale=720:1280:force_original_aspect_ratio=decrease[fg];
[bg][fg]overlay=(W-w)/2:(H-h)/2[vid]"
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
```

---

## 🔧 Requirements
- FFmpeg
- Python 3.11+
- faster-whisper (`pip install faster-whisper`)
- Poppins font installed

---

## 📝 License
Free to use for Hermes Agent workflows.
