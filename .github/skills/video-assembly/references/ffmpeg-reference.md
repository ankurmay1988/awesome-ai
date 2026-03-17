# FFmpeg Quick Reference for Video Assembly

## Essential Command Patterns

### Media Information
```bash
# Basic info
ffprobe -v error -show_entries format=duration,size,bit_rate -of default=noprint_wrappers=1 "file.mp4"

# Stream details
ffprobe -v error -show_entries stream=codec_name,codec_type,width,height,r_frame_rate,sample_rate,channels -of default=noprint_wrappers=1 "file.mp4"

# JSON output (full details)
ffprobe -v quiet -print_format json -show_format -show_streams "file.mp4"
```

### Trimming/Extraction
```bash
# Fast seek + copy (no re-encode)
ffmpeg -ss 00:00:30 -i "input.mp4" -t 15 -c copy -y "output.mp4"

# Extract from timestamp to timestamp
ffmpeg -ss 00:01:30 -to 00:02:45 -i "input.mp4" -c copy -y "output.mp4"

# Extract with re-encoding (more accurate)
ffmpeg -i "input.mp4" -ss 00:00:30 -t 15 -c:v libx264 -c:a aac -y "output.mp4"
```

### Image to Video Conversion
```bash
# Basic image to video (specific duration)
ffmpeg -loop 1 -i "image.jpg" -t 5 -c:v libx264 -pix_fmt yuv420p -r 30 -y "output.mp4"

# Image with resolution/padding (letterbox to 1920x1080)
ffmpeg -loop 1 -i "image.jpg" -t 5 -c:v libx264 -preset medium -crf 18 -pix_fmt yuv420p -vf "scale=1920:1080:force_original_aspect_ratio=decrease,pad=1920:1080:(ow-iw)/2:(oh-ih)/2" -r 30 -y "output.mp4"

# High-quality image video (for final production)
ffmpeg -loop 1 -i "image.png" -t 10 -c:v libx264 -preset slow -crf 18 -pix_fmt yuv420p -vf "scale=1920:1080:force_original_aspect_ratio=decrease,pad=1920:1080:(ow-iw)/2:(oh-ih)/2:color=black" -r 30 -y "image_video.mp4"

# Multiple images with specific durations (concat)
# Create image videos first, then use concat demuxer
ffmpeg -loop 1 -i "img1.jpg" -t 3 -c:v libx264 -pix_fmt yuv420p -r 30 -y "img1.mp4"
ffmpeg -loop 1 -i "img2.jpg" -t 5 -c:v libx264 -pix_fmt yuv420p -r 30 -y "img2.mp4"
# Then concat using list file
```

### Speed Adjustment
```bash
# Video only (slow down by 2x)
ffmpeg -i "input.mp4" -filter:v "setpts=2.0*PTS" -an -y "output.mp4"

# Video only (speed up by 2x)
ffmpeg -i "input.mp4" -filter:v "setpts=0.5*PTS" -an -y "output.mp4"

# Video + audio together (2x faster)
ffmpeg -i "input.mp4" -filter_complex "[0:v]setpts=0.5*PTS[v];[0:a]atempo=2.0[a]" -map "[v]" -map "[a]" -y "output.mp4"
```

### Audio Mixing
```bash
# Replace audio
ffmpeg -i "video.mp4" -i "audio.m4a" -c:v copy -c:a aac -b:a 192k -map 0:v:0 -map 1:a:0 -shortest -y "output.mp4"

# Mix two audio tracks together
ffmpeg -i "video.mp4" -i "audio.m4a" -filter_complex "[0:a][1:a]amix=inputs=2:duration=shortest[a]" -map 0:v -map "[a]" -c:v copy -c:a aac -y "output.mp4"

# Extract audio only (simple - see "Audio-Only Extraction" for more formats)
ffmpeg -i "video.mp4" -vn -c:a copy -y "audio.m4a"
```

### Concatenation
```bash
# Create list file (Windows PowerShell)
@"
file 'clip1.mp4'
file 'clip2.mp4'
file 'clip3.mp4'
"@ | Out-File -FilePath "list.txt" -Encoding utf8

# Concat with stream copy
ffmpeg -f concat -safe 0 -i "list.txt" -c copy -y "merged.mp4"

# Concat with re-encoding
ffmpeg -f concat -safe 0 -i "list.txt" -c:v libx264 -c:a aac -y "merged.mp4"
```

### Audio Normalization
```bash
# EBU R128 loudness (single pass)
ffmpeg -i "input.mp4" -c:v copy -af "loudnorm=I=-16:TP=-1.5:LRA=11" -c:a aac -b:a 192k -y "output.mp4"

# Two-pass for maximum accuracy (analysis)
ffmpeg -i "input.mp4" -af "loudnorm=I=-16:TP=-1.5:LRA=11:print_format=summary" -f null -

# Simple volume adjustment (+5dB)
ffmpeg -i "input.mp4" -c:v copy -af "volume=5dB" -c:a aac -y "output.mp4"
```

### Duration Control
```bash
# Trim to exact duration (keep beginning)
ffmpeg -i "input.mp4" -t 60 -c copy -y "output.mp4"

# Speed up video+audio together (1.5x faster)
ffmpeg -i "input.mp4" -filter_complex "[0:v]setpts=PTS/1.5[v];[0:a]atempo=1.5[a]" -map "[v]" -map "[a]" -c:v libx264 -c:a aac -y "output.mp4"

# Slow down video+audio together (1.5x slower)
ffmpeg -i "input.mp4" -filter_complex "[0:v]setpts=PTS*1.5[v];[0:a]atempo=0.667[a]" -map "[v]" -map "[a]" -c:v libx264 -c:a aac -y "output.mp4"

# Add black padding at end (extend duration)
ffmpeg -i "input.mp4" -vf "tpad=stop_mode=add:stop_duration=10" -af "apad=pad_dur=10" -y "output.mp4"

# Clone last frame at end (freeze frame extension)
ffmpeg -i "input.mp4" -vf "tpad=stop_mode=clone:stop_duration=10" -af "apad=pad_dur=10" -y "output.mp4"

# Fade to black + extend
ffmpeg -i "input.mp4" -filter_complex "[0:v]fade=t=out:st=38:d=2,tpad=stop_mode=add:stop_duration=20[v];[0:a]afade=t=out:st=38:d=2,apad=pad_dur=20[a]" -map "[v]" -map "[a]" -y "output.mp4"
```

### Resolution & Scaling
```bash
# Scale to 1920x1080 (may distort)
ffmpeg -i "input.mp4" -vf "scale=1920:1080" -c:a copy -y "output.mp4"

# Scale maintaining aspect ratio (fit 1920 width)
ffmpeg -i "input.mp4" -vf "scale=1920:-2" -c:a copy -y "output.mp4"

# Scale with padding to 16:9
ffmpeg -i "input.mp4" -vf "scale=1920:1080:force_original_aspect_ratio=decrease,pad=1920:1080:(ow-iw)/2:(oh-ih)/2" -c:a copy -y "output.mp4"
```

### Format Conversion
```bash
# MP4 to WebM
ffmpeg -i "input.mp4" -c:v libvpx-vp9 -c:a libopus -y "output.webm"

# Any format to MP4 (H.264 + AAC)
ffmpeg -i "input.avi" -c:v libx264 -preset medium -crf 23 -c:a aac -b:a 192k -y "output.mp4"

# Extract frames as images
ffmpeg -i "input.mp4" -vf "fps=1" "frame_%04d.png"
```

### Audio-Only Extraction
```bash
# Extract audio as MP3 (most compatible)
ffmpeg -i "video.mp4" -vn -c:a libmp3lame -b:a 192k -y "audio.mp3"

# Extract audio as AAC/M4A (high quality)
ffmpeg -i "video.mp4" -vn -c:a aac -b:a 192k -y "audio.m4a"

# Extract audio as WAV (uncompressed, lossless)
ffmpeg -i "video.mp4" -vn -c:a pcm_s16le -y "audio.wav"

# Extract audio as FLAC (lossless compressed)
ffmpeg -i "video.mp4" -vn -c:a flac -y "audio.flac"

# Extract audio as OGG Vorbis (open source)
ffmpeg -i "video.mp4" -vn -c:a libvorbis -q:a 6 -y "audio.ogg"

# Extract audio as Opus (modern, efficient)
ffmpeg -i "video.mp4" -vn -c:a libopus -b:a 128k -y "audio.opus"

# Extract audio keeping original codec (fastest)
ffmpeg -i "video.mp4" -vn -c:a copy -y "audio.m4a"

# Extract multiple formats at once (single read, efficient)
ffmpeg -i "video.mp4" -vn -c:a libmp3lame -b:a 192k "audio.mp3" -vn -c:a aac -b:a 192k "audio.m4a"
```

**Key Audio Parameters:**
- `-vn`: **Disable video** (audio-only output)
- `-c:a copy`: Keep original audio codec (fastest)
- `-c:a libmp3lame`: Convert to MP3
- `-c:a aac`: Convert to AAC
- `-c:a pcm_s16le`: Convert to WAV (uncompressed)
- `-c:a flac`: Convert to FLAC (lossless)
- `-c:a libvorbis`: Convert to OGG Vorbis
- `-c:a libopus`: Convert to Opus
- `-b:a 192k`: Audio bitrate (64k-320k)
- `-q:a 6`: Quality scale for OGG (0-10, 6 = ~192kbps)

### Video-Only Extraction (Silent)
```bash
# Extract video without audio - stream copy (fastest)
ffmpeg -i "video.mp4" -an -c:v copy -y "video_silent.mp4"

# Extract video to WebM (VP9, web-optimized)
ffmpeg -i "video.mp4" -an -c:v libvpx-vp9 -crf 30 -b:v 0 -y "video_silent.webm"

# Extract video to MKV container
ffmpeg -i "video.mp4" -an -c:v copy -y "video_silent.mkv"

# Re-encode video without audio (H.264)
ffmpeg -i "video.mp4" -an -c:v libx264 -preset medium -crf 23 -y "video_silent.mp4"

# Create GIF from video (no audio anyway)
ffmpeg -i "video.mp4" -vf "fps=10,scale=480:-1:flags=lanczos" -y "video.gif"
```

**Key Video Parameters:**
- `-an`: **Disable audio** (video-only output)
- `-c:v copy`: Keep original video codec (fastest)
- `-c:v libx264`: Convert to H.264
- `-c:v libvpx-vp9`: Convert to VP9 (WebM)
- `-crf 23`: Quality (0=lossless, 51=worst, 18-23 recommended)
- `-b:v 0`: Use CRF mode for VP9 (required for CRF)

### Cleanup Temporary Files
```powershell
# Remove specific patterns (safe)
Remove-Item "clip*_trimmed.*", "clip*_synced.*", "clip*_final.*", "*_video.mp4", "audio*.m4a", "final_merged.*", "concat_list.txt" -ErrorAction SilentlyContinue

# Preview what would be deleted (-WhatIf flag)
Remove-Item "*_trimmed.*", "*_synced.*", "*_video.mp4" -WhatIf

# Interactive cleanup - review first
Get-ChildItem | Where-Object { 
    $_.Name -match "(trimmed|synced|_final|_video|merged|concat)" -and
    $_.Name -notmatch "final_ready"
} | Format-Table Name, Length, LastWriteTime

# Then confirm and delete
Get-ChildItem | Where-Object { 
    $_.Name -match "(trimmed|synced|_final|_video|merged|concat)" -and
    $_.Name -notmatch "final_ready"
} | Remove-Item -Force

# Move final output and clean working directory
Move-Item "final_ready.mp4" "../final_ready.mp4"
Remove-Item "*" -Exclude "*.md", "*.txt"
```

## Windows Path Handling

```powershell
# Paths with spaces - use quotes
ffmpeg -i "C:\Users\John Doe\Videos\clip.mp4" -c copy -y "output.mp4"

# Escape special characters in PowerShell
$input = "C:\Users\ankurmathur01\Desktop\file.mp4"
ffmpeg -i "$input" -c copy -y "output.mp4"
```

## Timeout Recommendations

| Operation | Typical Duration | Recommended Timeout |
|-----------|-----------------|---------------------|
| Get info (ffprobe) | <1s | 5,000ms |
| Trim with copy | 1-5s | 30,000ms |
| Re-encode short clip | 10-60s | 120,000ms |
| Speed adjustment | 15-120s | 180,000ms |
| Audio mix | 5-15s | 60,000ms |
| Concat (copy) | 1-5s | 30,000ms |
| Normalize audio | 10-120s | 180,000ms |
| Duration control (trim) | 1-5s | 30,000ms |
| Duration control (speed) | 15-180s | 240,000ms |

## Speed Factor Calculations

### Video Speed (setpts filter)
```
Faster (compress time):  setpts=PTS/FACTOR
Slower (expand time):    setpts=PTS*FACTOR

Example - Speed up by 1.5x:  setpts=PTS/1.5
Example - Slow down by 1.5x: setpts=PTS*1.5
```
| "atempo out of range" | Speed > 2x or < 0.5x | Chain multiple atempo filters |
| "Requested value not available" | Incompatible filter combination | Check filter order and parameters |

### Audio Speed (atempo filter)
```
Faster: atempo=FACTOR (where FACTOR > 1.0)
Slower: atempo=FACTOR (where FACTOR < 1.0)

Limits: 0.5 ≤ FACTOR ≤ 2.0

For speeds outside range, chain filters:
atempo=2.0,atempo=1.25  →  2.5x faster total
atempo=0.5,atempo=0.5   →  0.25x slower (4x slower)
```

### Relationship Between Video and Audio
```
If video setpts=PTS/1.5 (1.5x faster)
Then audio atempo=1.5 (1.5x faster)

If video setpts=PTS*1.5 (1.5x slower)
Then audio atempo=0.667 (1/1.5 = 0.667, slower)
```

## Common Error Messages & Fixes

| Error | Cause | Solution |
|-------|-------|----------|
| "Non-monotonous DTS" | Incompatible streams in concat | Re-encode all clips to same format |
| "Protocol not found" | Path issue | Use absolute paths with proper escaping |
| "Invalid duration" | Bad timestamp format | Use HH:MM:SS or decimal seconds |
| "Output file is empty" | Codec mismatch | Specify codecs explicitly with `-c:v` `-c:a` |
| "Could not find codec" | Missing encoder | Use alternative codec or check ffmpeg build |
| Timeout/hanging | Large file processing | Use smaller timeout, cancel and retry |
| "Not a JPEG file" | Image format issue | Convert to supported format (JPEG/PNG) first |
| "Invalid pixel format" | Missing `-pix_fmt yuv420p` | Add pixel format for image-to-video |

## Image-to-Video Parameters

### Resolution Scaling Options
```bash
# Stretch to exact size (may distort)
-vf "scale=1920:1080"

# Fit inside dimensions (maintain aspect ratio)
-vf "scale=1920:1080:force_original_aspect_ratio=decrease"

# Fit and pad with black bars (letterbox/pillarbox)
-vf "scale=1920:1080:force_original_aspect_ratio=decrease,pad=1920:1080:(ow-iw)/2:(oh-ih)/2"

# Fit and pad with custom color
-vf "scale=1920:1080:force_original_aspect_ratio=decrease,pad=1920:1080:(ow-iw)/2:(oh-ih)/2:color=white"
```

### Duration Calculation for Images
```
If timestamps are: 00:00:05 to 00:00:12
Duration = 12 - 5 = 7 seconds
Command: ffmpeg -loop 1 -i "image.jpg" -t 7 ...
```

### Common Image Formats
- **JPEG/JPG**: Best for photos (lossy compression)
- **PNG**: Best for graphics, text, transparency (lossless)
- **BMP**: Uncompressed (large files)
- **TIFF**: High quality (may need conversion)

All formats work, but JPEG/PNG are most reliable.

## Quality vs Speed Matrix

### Video Encoding Presets
- **ultrafast**: 10x realtime, large files, preview quality
- **fast**: 5x realtime, medium files, good quality
- **medium**: 2-3x realtime, balanced (default)
- **slow**: 1x realtime, smaller files, better quality
- **veryslow**: 0.5x realtime, smallest files, best quality

### CRF Values (Constant Rate Factor)
- **0**: Lossless (huge files)
- **18**: Near-lossless, visually transparent
- **23**: Default, high quality
- **28**: Medium quality, noticeable compression
- **51**: Worst quality (minimum)

### Audio Bitrates
- **320 kbps**: Maximum quality (music)
- **192 kbps**: High quality (voice + music)
- **128 kbps**: Good quality (voice)
- **96 kbps**: Acceptable (podcasts)
- **64 kbps**: Minimum (speech only)
