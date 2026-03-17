# Output Format Guide - Audio-Only and Video-Only Extraction

This guide covers extracting audio-only or video-only from assembled video, and converting to specific output formats.

## When to Use

- User requests **audio-only** output (podcast, music, voiceover)
- User requests **video-only** output (silent video, background footage)
- User specifies a **specific format** (MP3, WAV, WebM, etc.)
- Converting final output to a **different container format**

## Key FFmpeg Parameters

### Stream Selection Options

| Option | Effect | Use Case |
|--------|--------|----------|
| `-vn` | **Disable video** - no video stream in output | Audio-only extraction |
| `-an` | **Disable audio** - no audio stream in output | Video-only (silent) |
| `-sn` | **Disable subtitles** - no subtitle stream | Remove subtitles |
| `-c copy` | **Stream copy** - no re-encoding | Fast, lossless transfer |
| `-c:a CODEC` | Specify audio codec | Convert audio format |
| `-c:v CODEC` | Specify video codec | Convert video format |
| `-f FORMAT` | Force output format | Override auto-detection |

### Audio-Only Output

Extract audio from video, discarding the video stream:

```bash
# Extract audio (keep original codec - fastest)
ffmpeg -i "video.mp4" -vn -c:a copy -y "audio.m4a"

# Extract and convert to MP3 (most compatible)
ffmpeg -i "video.mp4" -vn -c:a libmp3lame -b:a 192k -y "audio.mp3"

# Extract and convert to AAC (high quality, small size)
ffmpeg -i "video.mp4" -vn -c:a aac -b:a 192k -y "audio.aac"

# Extract and convert to WAV (uncompressed, high quality)
ffmpeg -i "video.mp4" -vn -c:a pcm_s16le -y "audio.wav"

# Extract and convert to FLAC (lossless compression)
ffmpeg -i "video.mp4" -vn -c:a flac -y "audio.flac"

# Extract and convert to OGG Vorbis (open source)
ffmpeg -i "video.mp4" -vn -c:a libvorbis -q:a 6 -y "audio.ogg"

# Extract and convert to Opus (modern, efficient)
ffmpeg -i "video.mp4" -vn -c:a libopus -b:a 128k -y "audio.opus"
```

### Video-Only Output

Extract video from video file, discarding the audio stream:

```bash
# Extract video (keep original codec - fastest)
ffmpeg -i "video.mp4" -an -c:v copy -y "video_silent.mp4"

# Extract and convert to WebM (VP9)
ffmpeg -i "video.mp4" -an -c:v libvpx-vp9 -crf 30 -b:v 0 -y "video_silent.webm"

# Extract to MKV container (flexible)
ffmpeg -i "video.mp4" -an -c:v copy -y "video_silent.mkv"

# Re-encode to H.264 (silent video)
ffmpeg -i "video.mp4" -an -c:v libx264 -preset medium -crf 23 -y "video_silent.mp4"

# Create GIF (no audio anyway)
ffmpeg -i "video.mp4" -vf "fps=10,scale=480:-1:flags=lanczos" -y "video.gif"
```

## Audio Format Comparison

| Format | Extension | Codec | Quality | File Size | Use Case |
|--------|-----------|-------|---------|-----------|----------|
| **MP3** | .mp3 | libmp3lame | Good | Small | Universal compatibility |
| **AAC** | .aac/.m4a | aac | Better | Small | Apple/mobile devices |
| **WAV** | .wav | pcm_s16le | Lossless | Very Large | Professional editing |
| **FLAC** | .flac | flac | Lossless | Large | Archival, audiophile |
| **OGG** | .ogg | libvorbis | Good | Small | Open source/web |
| **Opus** | .opus | libopus | Excellent | Very Small | Modern web/VoIP |

### Audio Quality Settings

**MP3 Bitrates:**
```bash
-b:a 320k  # Maximum quality (320 kbps)
-b:a 256k  # Very high quality
-b:a 192k  # High quality (recommended)
-b:a 128k  # Good quality (voice)
-b:a 96k   # Acceptable (podcasts)
-b:a 64k   # Minimum (speech only)
```

**AAC Bitrates:**
```bash
-b:a 256k  # Maximum quality
-b:a 192k  # High quality (recommended)
-b:a 128k  # Good quality
-b:a 96k   # Acceptable
-b:a 64k   # Low quality
```

**OGG Quality Scale (0-10):**
```bash
-q:a 10    # Maximum quality (~500 kbps)
-q:a 6     # High quality (~192 kbps) - recommended
-q:a 4     # Good quality (~128 kbps)
-q:a 2     # Acceptable (~96 kbps)
```

**Opus Bitrates:**
```bash
-b:a 256k  # Maximum quality
-b:a 128k  # High quality (recommended for music)
-b:a 64k   # Good quality (voice)
-b:a 32k   # Minimum (VoIP)
```

## Video Format Comparison

| Format | Extension | Codec | Quality | Size | Compatibility |
|--------|-----------|-------|---------|------|---------------|
| **MP4/H.264** | .mp4 | libx264 | Excellent | Medium | Universal |
| **MP4/H.265** | .mp4 | libx265 | Excellent | Small | Newer devices |
| **WebM/VP9** | .webm | libvpx-vp9 | Excellent | Small | Web browsers |
| **MKV** | .mkv | any | Any | Varies | Flexible container |
| **AVI** | .avi | varies | Varies | Large | Legacy systems |
| **GIF** | .gif | N/A | Limited | Large | Simple animations |

### Video Quality Settings

**CRF Values (H.264/H.265/VP9):**
```bash
-crf 0     # Lossless (huge files)
-crf 18    # Near-lossless, visually transparent
-crf 23    # Default, high quality
-crf 28    # Medium quality, smaller files
-crf 35    # Low quality, very small files
-crf 51    # Worst quality
```

## Complete Workflow Examples

### Example 1: Video Assembly → MP3 Audio Only

```bash
# After completing video assembly...
# Extract final audio as MP3
ffmpeg -i "final_ready.mp4" -vn -c:a libmp3lame -b:a 192k -y "final_audio.mp3"
```

### Example 2: Video Assembly → Multiple Audio Formats

```bash
# Extract in multiple formats at once (efficient single read)
ffmpeg -i "final_ready.mp4" \
    -vn -c:a libmp3lame -b:a 192k "output.mp3" \
    -vn -c:a aac -b:a 192k "output.m4a" \
    -vn -c:a flac "output.flac"
```

### Example 3: Video Assembly → Silent WebM for Web

```bash
# After video assembly, create web-optimized silent video
ffmpeg -i "final_ready.mp4" -an -c:v libvpx-vp9 -crf 30 -b:v 0 -y "final_silent.webm"
```

### Example 4: Video Assembly → Both Audio and Video Separately

```bash
# Extract audio
ffmpeg -i "final_ready.mp4" -vn -c:a copy -y "final_audio.m4a"

# Extract video (silent)
ffmpeg -i "final_ready.mp4" -an -c:v copy -y "final_video.mp4"
```

### Example 5: Convert Complete Video to Different Format

```bash
# MP4 to WebM (full re-encode)
ffmpeg -i "final_ready.mp4" -c:v libvpx-vp9 -crf 30 -b:v 0 -c:a libopus -b:a 128k -y "final_ready.webm"

# MP4 to MKV (just change container, no re-encode)
ffmpeg -i "final_ready.mp4" -c copy -y "final_ready.mkv"

# MP4 to AVI (legacy compatibility)
ffmpeg -i "final_ready.mp4" -c:v mpeg4 -q:v 3 -c:a mp3 -b:a 192k -y "final_ready.avi"
```

## User Confirmation Prompts

When user requests audio-only or video-only:

**Audio-Only Request:**
```
User requests: "Only audio" / "Extract audio" / "MP3 format"

Confirm:
1. Output format: [MP3/AAC/WAV/FLAC/OGG/OPUS]
2. Quality/bitrate: [192k recommended for MP3/AAC]
3. Output filename: [default: same as video with audio extension]

Proceed? [Y/N]
```

**Video-Only Request:**
```
User requests: "Only video" / "Silent video" / "No audio"

Confirm:
1. Output format: [MP4/WebM/MKV]  
2. Keep original codec or re-encode: [copy/re-encode]
3. Output filename: [default: same name with _silent suffix]

Proceed? [Y/N]
```

**Format Conversion Request:**
```
User requests: "Convert to WebM" / "Need MP3"

Confirm:
1. Input: current final video
2. Output format: [specified format]
3. Quality settings: [default/custom]
4. Re-encoding required: [Yes/No]
5. Estimated processing time: [based on duration]

Proceed? [Y/N]
```

## Decision Flow

```
User Request Analysis:
├── "audio only" / "extract audio" / "MP3" / "WAV" / "AAC"
│   └── → Audio-Only Extraction
│       ├── Determine format from keywords or ask
│       ├── Apply quality settings
│       └── Extract with -vn flag
│
├── "video only" / "silent" / "no audio" / "mute"
│   └── → Video-Only Extraction  
│       ├── Determine if re-encode needed
│       ├── Keep video codec if possible
│       └── Extract with -an flag
│
├── "convert to [FORMAT]" / "[FORMAT] format"
│   └── → Format Conversion
│       ├── Full video: convert both streams
│       ├── Audio only: extract + convert
│       └── Video only: extract + convert
│
└── No format specified
    └── → Default: MP4 with H.264 + AAC
```

## Common Errors and Solutions

| Error | Cause | Solution |
|-------|-------|----------|
| "Encoder not found" | Missing codec library | Use alternative codec or check ffmpeg build |
| "No audio stream" | Input has no audio | Cannot extract what doesn't exist |
| "Invalid sample rate" | Codec limitation | Re-sample audio: `-ar 44100` |
| "Container doesn't support codec" | Format mismatch | Use compatible codec for container |
| "Output file empty" | Filter/codec issues | Check logs, use explicit codec settings |

### Codec Availability Check

```bash
# Check if MP3 encoder is available
ffmpeg -encoders | findstr mp3

# Check if VP9 encoder is available  
ffmpeg -encoders | findstr vpx

# List all audio encoders
ffmpeg -encoders | findstr "^.A"

# List all video encoders
ffmpeg -encoders | findstr "^.V"
```

## File Size Estimates

### Audio (per minute)
| Format | Bitrate | Size/minute |
|--------|---------|-------------|
| MP3 128k | 128 kbps | ~0.96 MB |
| MP3 192k | 192 kbps | ~1.44 MB |
| MP3 320k | 320 kbps | ~2.4 MB |
| AAC 192k | 192 kbps | ~1.44 MB |
| WAV 16-bit | 1411 kbps | ~10.6 MB |
| FLAC | variable | ~3-6 MB |

### Video (per minute, 1080p)
| Format | CRF/Bitrate | Size/minute |
|--------|-------------|-------------|
| H.264 CRF 18 | variable | ~50-80 MB |
| H.264 CRF 23 | variable | ~25-40 MB |
| H.264 CRF 28 | variable | ~15-25 MB |
| VP9 CRF 30 | variable | ~20-35 MB |
| GIF | N/A | ~50-200 MB |

## Integration with Video Assembly Workflow

This step comes **after** the final video is assembled and verified:

```
Step 1-9: Full video assembly workflow
    ↓
Step 10: Output Format Conversion (if requested)
    ├── Audio-only extraction
    ├── Video-only extraction
    └── Format conversion
    ↓
Final deliverable in requested format
```
