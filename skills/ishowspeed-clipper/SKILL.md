---
name: ishowspeed-clipper
description: "Clipper pipeline untuk iShowSpeed YouTube channel. Topic 10158 - automated clip production. Viral segments: Speed Goes Pro (Tom Brady, Randy Orton, Kevin Durant), WWE, World Cup. 2 videos/day, auto-replenish."
tags: [clipper, youtube, ishowspeed, speedgoespro, topic-10158]
version: 1.0
---

# iShowSpeed Clipper - Topic 10158

**Channel:** https://www.youtube.com/@IShowSpeed
**Repliz Account:** 6a941f5b62caae1e040aae09
**Topic:** 10158 - Clipper System

## Viral Priority Queue

### Tier 1 (Speed Goes Pro - Highest Viral)
| Video ID | Title | Priority |
|----------|-------|----------|
| 5ZKLMLm2YLs | SPEED GOES PRO EP 1: TOM BRADY | ✅ Done |
| pySIRc4QjsY | SPEED GOES PRO EP 5: RANDY ORTON (WWE) | Next |
| v9J2Lwzjbac | SPEED GOES PRO EP 4: KEVIN DURANT | Queue |
| 96l2Rmr2Mso | SPEED GOES PRO EP 3: JOEY CHESTNUT | Queue |
| 2tETVaG_mZI | SPEED GOES PRO EP 2: SUNI LEE | Queue |

### Tier 2 (High Viral)
| Video ID | Title |
|----------|-------|
| KU9jGVMlLV0 | How I Fought In WWE WrestleMania 42 |
| fMUp9cb6b5Y | How I Performed At The World Cup Finals! |
| 5hTAg2ThHAo | I Spent 30 Days Exploring All Of Africa! |
| VT3CtCEh8HQ | I Spent 14 Days Exploring The Caribbean! |

### Tier 3 (Music Videos)
| Video ID | Title |
|----------|-------|
| vrY1THC_NQE | World Cup (Champions) |
| JJ-Dyoglslg | Speed Gang Anthem |

## Workflow

### Auto-Produce Daily (2 videos)
1. Pick next video from priority queue
2. Download video
3. Transcribe (faster-whisper)
4. Find viral segment (high energy, controversial, celebrity)
5. Generate 59s clip
6. Upload to VPS
7. Schedule to Repliz
8. Update used videos list

### Production Script
```bash
cd /home/admin/clipper-company
python3 ishowspeed_producer.py
```

## Subtitle Settings (Same as Indigenous Australians v26)
- Subtitle: Poppins Bold 65px, white, MarginV=100
- Source: Poppins Bold 36px, yellow, top
- Background: Cinematic blur (same FFmpeg filter)

## Schedule
- 2 videos/day to Repliz
- Auto-replenish queue when < 2 videos remaining
- Cron: production trigger every 12 hours

## Output
- VPS: /home/admin/domains/digitalnusa.com/public_html/anime-red/videos/
- URL: https://digitalnusa.com/anime-red/videos/ishowspeed_v{n}.mp4
