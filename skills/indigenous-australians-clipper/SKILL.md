---
name: indigenous-australians-clipper
description: "Clipper pipeline untuk Indigenous Australians documentary (ABC Australia). YouTube: https://youtu.be/MwLuPhwpViI. FFmpeg: cinematic blur bg, letterbox 9:16, ASS subtitle Gen-Z style."
tags: [clipper, youtube, indigenous-australians, documentary, abc-australia, gen-z-subtitle]
version: 1.0
---

# Indigenous Australians Documentary — Clipper Pipeline

**Source:** https://youtu.be/MwLuPhwpViI
**Channel:** ABC Australia — "You Can't Ask That"
**Topic:** Indigenous Australians documentary

## Specs

- **Output:** 9:16 vertical 720×1280
- **Background:** Cinematic blur
  - Scale 1280x720 → split → bg: increase+crop+boxblur, fg: decrease → overlay
- **Subtitle:** Poppins Bold 65px, white, bottom letterbox area
  - MarginV: 100 (position in letterbox)
  - Outline: 3px, Shadow: 1.5px
- **Source credit:** Poppins Bold 36px, yellow (#FFFF00), top center
- **Hook:** 40px bold white with black shadow

## FFmpeg Filter

```bash
Vf="scale=1280:720:flags=lanczos,setsar=1[base];
[base]split=2[bga][fga];
[bga]scale=720:1280:force_original_aspect_ratio=increase,crop=720:1280,boxblur=15:2[bg];
[fga]scale=720:1280:force_original_aspect_ratio=decrease[fg];
[bg][fg]overlay=(W-w)/2:(H-h)/2[vid]"
```

## Workflow Steps

### 1. Download Video
```bash
VIDEO_ID=MwLuPhwpViI
yt-dlp -f 18 \
  -o "downloads/${VIDEO_ID}_full.%(ext)s" \
  "https://youtube.com/watch?v=${VIDEO_ID}"
```

### 2. Transcribe
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

### 3. Generate Clip
```python
#!/usr/bin/env python3
"""Gen-Z Clip - Indigenous Australians"""
import json, subprocess, os

VIDEO_ID = "MwLuPhwpViI"
VIDEO = f"downloads/{VIDEO_ID}_full.mp4"
OUT_DIR = f"clips/{VIDEO_ID}/processed/v26"
os.makedirs(OUT_DIR, exist_ok=True)

with open('/tmp/transcript_full.json') as f:
    transcript = json.load(f)

CLIP_START = 172.0

def sec(ts):
    h = int(ts // 3600)
    m = int((ts % 3600) // 60)
    s = int(ts % 60)
    return f"{h}:{m:02d}:{s:02d}.{int((ts % 1) * 100):02d}"

def wrap_text(text, max_chars=40):
    words = text.split()
    lines, current = [], ""
    for w in words:
        if len(current) + len(w) + 1 <= max_chars:
            current = (current + " " + w).strip()
        else:
            if current: lines.append(current)
            current = w
    if current: lines.append(current)
    return lines

clip_segs = [s for s in transcript if s['end'] > CLIP_START and s['start'] < CLIP_START + 59]
clip_segs.sort(key=lambda x: x['start'])

merged = []
buffer_text = ""
buffer_start = None
for seg in clip_segs:
    text = seg['text'].strip()
    if not buffer_start:
        buffer_start = seg['start']
    buffer_text += " " + text
    buffer_text = buffer_text.strip()
    if len(buffer_text) > 55 or seg == clip_segs[-1]:
        merged.append({'start': buffer_start, 'end': seg['end'], 'text': buffer_text})
        buffer_text = ""
        buffer_start = None

ass = """[Script Info]
Title: GenZ Clip
ScriptType: v4.00+
WrapStyle: 0
ScaledBorderAndShadow: yes
PlayResX: 720
PlayResY: 1280

[V4+ Styles]
Format: Name,Fontname,Fontsize,PrimaryColour,SecondaryColour,OutlineColour,BackColour,Bold,Italic,Underline,StrikeOut,ScaleX,ScaleY,Spacing,Angle,BorderStyle,Outline,Shadow,Alignment,MarginL,MarginR,MarginV,Encoding
Style: Sub,Poppins Bold,65,&H00FFFFFF,&H000000FF,&H00000000,&HCC000000,-1,0,0,0,100,100,0,0,1,3,1.5,2,15,15,100,1
Style: Src,Poppins Bold,36,&H00FFFF00,&H000000FF,&H00000000,&H00000000,-1,0,0,0,100,100,0,0,1,2.5,2,8,10,10,20,1
Style: Hook,Poppins Bold,40,&H00FFFFFF,&H000000FF,&H00000000,&HAA000000,-1,0,0,0,100,100,2,0,1,3,2,2,10,10,1050,1
Style: Cta,Poppins SemiBold,20,&H00FFFFFF,&H000000FF,&H00000000,&HAA000000,-1,0,0,0,100,100,0,0,1,2,1,2,10,10,70,1

[Events]
Format: Layer,Start,End,Style,Name,MarginL,MarginR,MarginV,Effect,Text
"""

# Source credit
ass += f"Dialogue: 0,{sec(0)},{sec(59)},Src,,0,0,0,,ABC Australia | You Can't Ask That\n"
# Hook
ass += f"Dialogue: 1,{sec(0)},{sec(2.5)},Hook,,0,0,0,,WHAT FREE STUFF FROM GOVT?\n"

# Subtitles
for item in merged:
    start = item['start'] - CLIP_START
    end = item['end'] - CLIP_START
    if start < 0: start = 0
    if end > 59: end = 59
    if start >= 59: continue
    
    lines = wrap_text(item['text'], 40)
    seg_dur = (end - start) / len(lines) if lines else 2
    
    for i, line in enumerate(lines):
        line_start = start + (i * seg_dur)
        line_end = min(line_start + seg_dur + 0.3, end)
        ass += f"Dialogue: 1,{sec(line_start)},{sec(line_end)},Sub,,0,0,0,,{line}\n"

# CTA
ass += f"Dialogue: 0,{sec(52)},{sec(58)},Cta,,0,0,0,,Watch full episode on ABC iview\n"

ASS_FILE = f"{OUT_DIR}/clip.ass"
with open(ASS_FILE, 'w', encoding='utf-8') as f:
    f.write(ass)

OUTPUT = f"{OUT_DIR}/final.mp4"

cmd = [
    'ffmpeg', '-y',
    '-ss', str(CLIP_START),
    '-t', '59',
    '-i', VIDEO,
    '-filter_complex',
    '[0:v]scale=1280:720:flags=lanczos,setsar=1[base];'
    '[base]split=2[bga][fga];'
    '[bga]scale=720:1280:force_original_aspect_ratio=increase,crop=720:1280,boxblur=15:2[bg];'
    '[fga]scale=720:1280:force_original_aspect_ratio=decrease[fg];'
    '[bg][fg]overlay=(W-w)/2:(H-h)/2[vid];'
    '[vid]ass=' + ASS_FILE + '[out]',
    '-map', '[out]',
    '-map', '0:a',
    '-c:v', 'libx264',
    '-crf', '20',
    '-preset', 'fast',
    '-r', '30',
    '-c:a', 'aac',
    '-b:a', '128k',
    '-ar', '48000',
    '-pix_fmt', 'yuv420p',
    '-movflags', '+faststart',
    OUTPUT
]

subprocess.run(cmd)
print(f"Output: {OUTPUT}")
```

### 4. Upload to VPS
```bash
# Copy to your public VPS folder
cp "clips/${VIDEO_ID}/processed/v26/final.mp4" /var/www/html/videos/
```

## Repliz Schedule
```bash
AUTH=$(echo -n "$REPLIZ_API_KEY" | base64)
ACCOUNT_ID="${REPLIZ_ACCOUNT_ID}"
VIDEO_URL="https://your-domain.com/videos/${VIDEO_ID}_v26.mp4"

curl -X POST "https://api.repliz.com/public/schedule" \
  -H "Authorization: Basic ${AUTH}" \
  -H "Content-Type: application/json" \
  -d '{
    "accountId": "'${ACCOUNT_ID}'",
    "type": "video",
    "title": "What the government gives us...",
    "topic": "Entertainment",
    "medias": [{"type": "video", "url": "'${VIDEO_URL}'"}],
    "scheduleAt": "2026-09-01T14:00:00Z"
  }'
```

## Subtitle Settings

| Element | Font | Size | Color | Position |
|---------|------|------|-------|----------|
| Subtitle | Poppins Bold | 65px | White | Bottom letterbox (MarginV=100) |
| Source | Poppins Bold | 36px | Yellow | Top center |
| Hook | Poppins Bold | 40px | White | Center with shadow |
| CTA | Poppins SemiBold | 20px | White | Bottom |

## Clip Segment

- **Start:** 172.0s
- **Duration:** 59s
- **Hook:** "WHAT FREE STUFF FROM GOVT?"
- **Content:** Q&A about government benefits for Indigenous Australians
