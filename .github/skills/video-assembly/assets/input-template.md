# Video Assembly Input Template

Copy this template and fill in your video/audio chunks with timestamps.

## Project Info
- **Project Name**: [Your project name]
- **Output File**: [final_video.mp4]
- **Target Duration**: [Total seconds or HH:MM:SS] *(Optional - leave blank for auto)*
- **Master Timing Source**: [ ] Video  [X] Audio  [ ] None

**Duration Strategy** (if content doesn't match target):
- [ ] Speed adjust proportionally
- [ ] Trim content to fit
- [ ] Add padding/black frames
- [ ] Accept different duration
- [ ] Ask me before proceeding

---

## Video Clips

### Clip 1
- **Source File**: `[Path to video file]`
- **Timestamps**: `[START_TIME]` to `[END_TIME]`  
  *(If entire file, write "Full File")*
- **Expected Duration**: `[X seconds]`
- **Notes**: [Any special requirements]

### Clip 2
- **Source File**: `[Path to video file]`
- **Timestamps**: `[START_TIME]` to `[END_TIME]`
- **Expected Duration**: `[X seconds]`
- **Notes**: [Any special requirements]

### Clip 3
- **Source File**: `[Path to video file]`
- **Timestamps**: `[START_TIME]` to `[END_TIME]`
- **Expected Duration**: `[X seconds]`
- **Notes**: [Any special requirements]

*(Add more clips as needed)*

---

## Image Segments

### Image 1
- **Source File**: `[Path to image file (JPG/PNG/etc)]`
- **Display Duration**: `[START_TIME]` to `[END_TIME]`  
  *(Image will be shown in final video during this time range)*
- **Calculated Duration**: `[X seconds]` *(END - START)*
- **Target Resolution**: `[1920x1080 / Match video / Other]`
- **Notes**: [Any special requirements, e.g., "title card", "credits"]

### Image 2  
- **Source File**: `[Path to image file]`
- **Display Duration**: `[START_TIME]` to `[END_TIME]`
- **Calculated Duration**: `[X seconds]`
- **Target Resolution**: `[1920x1080 / Match video / Other]`
- **Notes**: [Any special requirements]

*(Add more images as needed, or remove this section if no images)*

---

## Audio Segments

### Audio for Clip 1
- **Source File**: `[Path to audio file]`
- **Timestamps**: `[START_TIME]` to `[END_TIME]`  
  *(If entire file, write "Full File")*
- **Expected Duration**: `[X seconds]`
- **Notes**: [Any special requirements]

### Audio for Clip 2
- **Source File**: `[Path to audio file]`
- **Timestamps**: `[START_TIME]` to `[END_TIME]`
- **Expected Duration**: `[X seconds]`
- **Notes**: [Any special requirements]

### Audio for Clip 3
- **Source File**: `[Path to audio file]`
- **Timestamps**: `[START_TIME]` to `[END_TIME]`
- **Expected Duration**: `[X seconds]`
- **Notes**: [Any special requirements]

*(Add more audio segments as needed)*

---

## Processing Options

- **Video Speed Adjustment**: [ ] Yes  [ ] No  
  *(Check Yes if video should stretch/shrink to match audio)*

- **Audio Normalization**: [ ] Yes  [ ] No  
  *(EBU R128 loudness normalization)*

- **Target Loudness**: `-16 LUFS` *(default for broadcast)*

- **Output Quality**:
  - CRF: `18` *(18=near-lossless, 23=default, 28=medium)*
  - Preset: `medium` *(ultrafast/fast/medium/slow/veryslow)*

- **Output Resolution**: `[Keep Original / 1920x1080 / Other]`

---

## Output Format Options

**Final Output Type** (check one):
- [X] Full video (video + audio) *(default)*
- [ ] Audio-only (extract audio from final video)
- [ ] Video-only (silent, no audio)

**If selecting Audio-Only, choose format:**
- [ ] MP3 - Universal compatibility (recommended)
- [ ] AAC/M4A - High quality, Apple-friendly
- [ ] WAV - Uncompressed, lossless (large files)
- [ ] FLAC - Lossless compressed (archival)
- [ ] OGG - Open source format
- [ ] Opus - Modern, efficient (web/streaming)

**Audio Quality** (if audio-only):
- Bitrate: `192k` *(64k-320k for MP3/AAC, ignored for WAV/FLAC)*

**If selecting Video-Only, choose format:**
- [ ] MP4 (H.264) - Keep original codec
- [ ] WebM (VP9) - Web-optimized
- [ ] MKV - Flexible container

**Alternative Output Formats** (in addition to or instead of MP4):
- [ ] Also create WebM version
- [ ] Also create MKV version
- [ ] Convert to different container: `[Specify format]`

---

- **Cleanup After Processing**: [ ] Yes  [ ] No  
  *(Automatically remove temporary files after verification)*

- **Quality Thresholds**:
  - Max acceptable speed change: `1.5x` *(Warn if exceeded)*
  - Minimum acceptable CRF: `28` *(Warn if quality too low)*
  - Allow multiple re-encodes: [ ] Yes  [ ] No

---

## Example: Filled Template

```markdown
- **Duration Strategy**: [X] Speed adjust proportionally
## Project Info
- **Project Name**: Mission Video Assembly
- **Output File**: final_mission_video.mp4
- **Target Duration**: 61 seconds
- **Master Timing Source**: [X] Audio

## Video Clips

### Clip 1
- **Source File**: `C:\Users\ankurmathur01\Desktop\Mission2.mp4`
- **Timestamps**: `00:00:11` to `00:00:26` (15 seconds)
- **Expected Duration**: 32 seconds (after speed adjustment)
- **Notes**: Will be slowed down to match audio

### Clip 2
- **Source File**: `C:\Users\ankurmathur01\Desktop\Mission3.mp4`
- **Timestamps**: `00:00:00` to `00:00:22` (22 seconds)
- **Expected Duration**: 29 seconds (after speed adjustment)
- **Notes**: Will be slowed down to match audio

## Image Segments

### Image 1
- **Source File**: `C:\Users\ankurmathur01\Desktop\title_card.jpg`
- **Display Duration**: `00:00:00` to `00:00:05` (5 seconds)
- **Calculated Duration**: 5 seconds
- **Target Resolution**: 1920x1080
- **Notes**: Opening title card

## Audio Segments

### Audio for Image 1
- **Source File**: `C:\Users\ankurmathur01\Desktop\recording.m4a`
- **Timestamps**: `00:00:00` to `00:00:05` (5 seconds)
- **Expected Duration**: 5 seconds
- **Notes**: Music for title

### Audio for Clip 1
- **Source File**: `C:\Users\ankurmathur01\Desktop\recording.m4a`
- **Timestamps**: `00:00:38` to `00:01:10` (32 seconds)
- **Expected Duration**: 32 seconds
- **Notes**: Extracted from full recording

### Audio for Clip 2
- **Source File**: `C:\Users\ankurmathur01\Desktop\recording.m4a`
- **Timestamps**: `00:01:11` to `00:01:40` (29 seconds)
- **Expected Duration**: 29 seconds
- **Notes**: Extracted from full recording

## Processing Options
- **Video Speed Adjustment**: [X] Yes
- **Audio Normalization**: [X] Yes
- **Target Loudness**: -16 LUFS
- **Output Quality**: CRF: 18, Preset: medium
- **Output Resolution**: Keep Original (1920x1080)
- **Quality Thresholds**: Max speed: 1.5x, Min CRF: 28
- **Cleanup After Processing**: [X] Yes
```

---
### Duration Calculation for Images
- Start: 00:00:05, End: 00:00:12 → Duration = 7 seconds
- Start: 00:00:00, End: 00:00:10 → Duration = 10 seconds
- Start: 00:01:30, End: 00:01:45 → Duration = 15 seconds


## Timestamp Format Reference

### Accepted Formats
- **Seconds**: `30`, `90.5`, `125`
- **MM:SS**: `01:30`, `02:05`
- **HH:MM:SS**: `00:01:30`, `01:23:45`
- **HH:MM:SS.MS**: `00:01:30.500`

### Quick Conversions
- 1 minute = 60 seconds
- 1:30 = 90 seconds
- 2:15 = 135 seconds
- 00:01:40 = 100 seconds

---

## Checklist Before Processing

- [ ] All file paths are correct and accessible
- [ ] All timestamps are in correct format
- [ ] Total video duration + total audio duration calculated
- [ ] Master timing source identified (audio or video)
- [ ] Speed adjustment requirements noted
- [ ] Expected output duration confirmed
- [ ] Quality settings chosen
- [ ] Output file path specified

## Checklist After Processing

- [ ] Final video plays correctly from start to finish
- [ ] Audio is present and in sync with video
- [ ] Duration matches expected length
- [ ] Quality is acceptable (no artifacts, proper resolution)
- [ ] File size is reasonable
- [ ] Final output saved to correct location
- [ ] Temporary files cleaned up (if desired)
- [ ] Original source files are still intact (never delete these!)
