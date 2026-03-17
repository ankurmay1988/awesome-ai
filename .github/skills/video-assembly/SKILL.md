---
name: video-assembly
description: 'Assemble final video from video/audio chunks with timestamps using ffmpeg. Use for: syncing video clips with voiceover, trimming media segments, combining multiple clips, adjusting video speed to match audio duration, merging separate or timestamped chunks into final output. Supports audio-only extraction (MP3/WAV/AAC/FLAC), video-only output, and format conversion. Handles both MCP tools and raw ffmpeg commands.'
argument-hint: 'Provide video clips, audio files, images, timestamps, and desired output format'
---

# Video Assembly with FFmpeg

Workflow for assembling a complete video from multiple video/audio/image chunks with precise timestamp control. Handles chunks from both separate files and timestamped segments within larger files.

## When to Use

- Syncing pre-cut video clips with voiceover audio
- Assembling videos from timestamped segments
- Combining multiple video/audio chunks into final output
- Adjusting video playback speed to match audio duration
- Creating videos where audio timing is the master (video conforms to audio)
- Inserting images as video segments with specific display durations
- Creating photo slideshows with audio
- Adding title cards, credits, or static images to videos

## Input Scenarios Supported

1. **Multiple separate files** (e.g., clip1.mp4, clip2.mp4, audio1.wav)
2. **Timestamped segments from single file** (e.g., BigVideo.mp4 → 00:01:30-00:02:45)
3. **Mixed sources** (some separate files, some timestamped extractions)
4. **Images with duration** (e.g., title.jpg → 00:00:00-00:00:05 = 5 second display)
5. **Any media type** with timestamps requiring trimming/isolation

## Core Workflow

**10-Step Process:** Analyze → Duration Check → Trim → Sync → Mix → Merge → Duration Control → Normalize → Verify → **Output Format** → Cleanup

### Step 1: Analyze All Media

Get information on every input file using `get_media_info` (ffmpeg-mcp) or raw ffprobe:

```bash
# MCP Tool (preferred)
mcp_ffmpeg-mcp_get_media_info

# Fallback: Raw ffprobe
ffprobe -v error -show_entries format=duration:stream=codec_name,width,height,r_frame_rate,sample_rate -of default=noprint_wrappers=1 "file.mp4"
```

**Report:**
- Duration of each clip/segment
- Total combined video duration
- Total audio duration
- Codec information
- Resolution and frame rate

### Step 2: Duration Check & Strategy Planning

**If user specified target duration**, compare with calculated total:

```
Total Content Duration: Sum of all clips/images/audio segments
Target Duration: User-specified final video length (if provided)

If Total > Target: Content is TOO LONG → Need to reduce
If Total < Target: Content is TOO SHORT → Options available
If Total ≈ Target: Perfect fit → Proceed normally
```

**Duration Over Target - Options:**

1. **Speed Up Proportionally** (Quality Impact: Low if <1.5x)
   ```
   Speed Factor = Target Duration / Total Duration
   Example: 80s content → 60s target = 0.75x speed (25% faster)
   
   Quality Threshold: Warn if speed > 1.5x (noticeable to viewers)
   ```

2. **Trim End Content** (Quality Impact: None, but loses content)
   ```
   Keep first N seconds/clips, discard rest
   User chooses what to cut
   ```

3. **Selective Trimming** (Quality Impact: None, surgical cuts)
   ```
   User specifies which clips to shorten or remove
   More control, better result
   ```

4. **Lossy Compression** (Quality Impact: Medium-High)
   ```
   Reduce bitrate to fit file size limits
   Not recommended for duration control
   ```

**Duratio4: Sync Video Speed to Audio (or Target Duration)

1. **Slow Down Proportionally** (Quality Impact: None)
   ```
   Speed Factor = Target Duration / Total Duration
   Example: 40s content → 60s target = 1.5x slower (50% slower)
   
   Quality Threshold: Warn if speed > 2.0x (very slow motion)
   ```

2. **Add Padding** (Quality Impact: None)
   ```
   Black frames, fade to black, or hold last frame
   ```

3. **Accept Shorter** (No Impact)
   ```
   Final video is shorter than target - may be acceptable
   ```

**Quality Thresholds:**
- ⚠️ **Speed change > 1.5x**: Noticeable, may look unnatural
- 🛑 **Speed change > 2.5x**: Very obvious, likely unacceptable
- ⚠️ **CRF > 28**: Visible compression artifacts
- ⚠️ **Multiple re-encodes**: Quality degradation

**For detailed strategies, see:** [Duration Management Guide](./references/duration-management.md)

### Step 3: Trim/Extract Segments & Convert Images

For any file with timestamps, extract the specific segment:

```bash
# Video/Audio Trimming
# MCP Tool (may timeout on large files)
mcp_ffmpeg-mcp_trim_video

# Fallback: Raw ffmpeg (RELIABLE)
ffmpeg -ss START_TIME -i "INPUT_FILE" -t DURATION -c copy -y "OUTPUT_FILE"
# Example: -ss 00:00:11 -t 15 extracts 15 seconds starting at 11s
```

For images with duration timestamps, convert to video segment:

```bash
# Convert image to video with specific duration
ffmpeg -loop 1 -i "image.jpg" -t DURATION -c:v libx264 -preset medium -crf 18 -pix_fmt yuv420p -vf "scale=1920:1080:force_original_aspect_ratio=decrease,pad=1920:1080:(ow-iw)/2:(oh-ih)/2" -r 30 -y "image_video.mp4"

# Example: Show title card for 5 seconds
ffmpeg -loop 1 -i "title.jpg" -t 5 -c:v libx264 -pix_fmt yuv420p -vf "scale=1920:1080:force_original_aspect_ratio=decrease,pad=1920:1080:(ow-iw)/2:(oh-ih)/2" -r 30 -y "title_video.mp4"
```

**Key Parameters:**
- **Video/Audio:**
  - `-ss START_TIME`: Start timestamp (HH:MM:SS or seconds)
  - `-t DURATION`: Duration to extract (seconds or HH:MM:SS)
  - `-c copy`: Fast stream copy (no re-encoding)
  - Always use `-ss` BEFORE `-i` for fast seeking

- **Images:**
  - `-loop 1`: Loop the image (required for still images)
  - `-t DURATION`: How long to display the image (calculated from end_time - start_time)
  - `-pix_fmt yuv420p`: Pixel format for compatibility
  - `-vf scale=...`: Scale and pad to match video resolution with letterboxing
  - `-r 30`: Frame rate (match your video clips)
  - Calculate DURATION from timestamp range: end_time - start_time

**For detailed image conversion options, see:** [Image-to-Video Guide](./references/image-to-video-guide.md)

### Step 3: Sync Video Speed to Audio

When audio is the master timing source OR user specified target duration, adjust video speed to match:

```bash
# MCP Tool: change_speed (may hang/timeout)
mcp_ffmpeg-mcp_change_speed

# Fallback: Raw ffmpeg with setpts filter (RELIABLE)
# To slow down by factor (e.g., 2.0x slower = half speed):
ffmpeg -i "input.mp4" -filter:v "setpts=FACTOR*PTS" -c:v libx264 -preset medium -crf 18 -an -y "output_synced.mp4"

# Calculate FACTOR:
# FACTOR = target_duration / current_duration
# Example: 15s video → 32s: FACTOR = 32/15 = 2.133

# For entire merged video (if adjusting after merge):
ffmpeg -i "merged.mp4" -filter_complex "[0:v]setpts=FACTOR*PTS[v];[0:a]atempo=TEMPO[a]" -map "[v]" -map "[a]" -c:v libx264 -preset medium -crf 18 -c:a aac -y "output_adjusted.mp4"

# TEMPO calculation (audio speed, limited to 0.5-2.0):
# TEMPO = 1/FACTOR (inverse of video speed)
# For speeds outside 0.5-2.0, chain multiple atempo filters
```

**Quality Warning Check:**
```powershell
$speedFactor = $targetDuration / $currentDuration
if ($speedFactor -gt 1.5 -or $speedFactor -lt 0.67) {
    Write-Warning "Speed change of ${speedFactor}x may be noticeable to viewers"
    # Prompt user for confirmation or alternative approach
}
if ($speedFactor -gt 2.5 -or $speedFactor -lt 0.4) {
    Write-Error "Speed change of ${speedFactor}x will be very obvious - consider trimming instead"
    # Offer alternative: trim content to target duration
}
```

**Parameters:**
- `setpts=FACTOR*PTS`: Time stretching (>1 = slower, <1 = faster)
- `-c:v libx264`: Re-encode video (required for speed change)
- `-preset medium`: Encoding speed (fast/medium/slow)
- `-crf 18`: Quality (lower = better, 18 = near-lossless)
- `-an`: Remove audio (will be added later)
6
### Step 4: Mix Audio into Video

Replace or add audio track to each video clip:

```bash
# MCP Tool: mix_audio (may fail)
mcp_ffmpeg-mcp_mix_audio

# Fallback: Raw ffmpeg (RELIABLE)
ffmpeg -i "video.mp4" -i "audio.m4a" -c:v copy -c:a aac -b:a 192k -map 0:v:0 -map 1:a:0 -shortest -y "output_final.mp4"
```

**Parameters:**
- `-map 0:v:0`: Use video from first input
- `-map 1:a:0`: Use audio from second input
- `-c:v copy`: Copy video stream (no re-encoding)
- `-c:a aac -b:a 192k`: Encode audio as AAC at 192kbps
- `-shortest`: End when shortest stream ends

### Step 5: Merge All Clips

Concatenate multiple video clips in order:

```bash
# Create 7: Duration Control (If Target Specified)

If final merged video doesn't match target duration exactly:

**Option A: Trim to Exact Duration**
```bash
# Trim merged video to target duration
ffmpeg -i "merged.mp4" -t TARGET_DURATION -c copy -y "trimmed.mp4"

# Example: Limit to 60 seconds
ffmpeg -i "merged.mp4" -t 60 -c copy -y "trimmed.mp4"
```

**Option B: Speed Adjust Entire Video**
```bash
# Adjust both video and audio speed together
ffmpeg -i "merged.mp4" -filter_complex "[0:v]setpts=PTS/SPEED[v];[0:a]atempo=SPEED[a]" -map "[v]" -map "[a]" -c:v libx264 -c:a aac -y "adjusted.mp4"

# Example: 70s video → 60s (speed up by 1.167x)
ffmpeg -i "merged.mp4" -filter_complex "[0:v]setpts=PTS/1.167[v];[0:a]atempo=1.167[a]" -map "[v]" -map "[a]" -c:v libx264 -preset medium -crf 18 -c:a aac -y "adjusted.mp4"
```

**Option C: Add Padding/Extend**
```bash
# Add black frames at end to reach target duration
ffmpeg -i "merged.mp4" -vf "tpad=stop_mode=clone:stop_duration=EXTRA_SECONDS" -c:a copy -y "padded.mp4"

# Example: 50s video → 60s (add 10s of last frame hold)
ffmpeg -i "merged.mp4" -vf "tpad=stop_mode=clone:stop_duration=10" -c:a copy -y "padded.mp4"
```

**Option D: Fade to Black + Extend**
```bash
# Fade out video and audio, then extend with black
ffmpeg -i "merged.mp4" -filter_complex "[0:v]fade=t=out:st=START_TIME:d=FADE_DURATION,tpad=stop_mode=black:stop_duration=EXTRA_SECONDS[v];[0:a]afade=t=out:st=START_TIME:d=FADE_DURATION,apad=pad_dur=EXTRA_SECONDS[a]" -map "[v]" -map "[a]" -y "extended.mp4"
```10

**User Confirmation Required When:**
- Speed change > 1.5x in either direction
- Trimming more than 10% of content
- Quality settings need to be lowered
- Re-encoding would be the 2nd+ time

### Step 8: Normalize Audio (Optional)
@"
file 'clip1_final.mp4'
file 'clip2_final.mp4'
file 'clip3_final.mp4'
"@ | Out-File -FilePath "concat_list.txt" -Encoding utf8

# MCP Tool: merge_videos (may timeout)
mcp_ffmpeg-mcp_merge_videos

# Fallback: Raw ffmpeg concat (RELIABLE)
ffmpeg -f concat -safe 0 -i "concat_list.txt" -c copy -y "merged.mp4"
```

**NOTE:** All clips must have same codec, resolution, and frame rate for `-c copy` to work. Otherwise, re-encode:

```bash
ffmpeg -f concat -safe 0 -i "concat_list.txt" -c:v libx264 -preset medium -crf 18 -c:a aac -b:a 192k -y "merged.mp4"
```

### Step 6: Normalize Audio (Optional)

Apply EBU R128 loudness normalization for consistent audio levels:

```bash
# MCP Tool: normalize_audio (may timeout)
mcp_ffmpeg-mcp_normalize_audio

# Fallback: Raw ffmpeg loudnorm (RELIABLE)
ffmpeg -i "merged.mp4" -c:v copy -af "loudnorm=I=-16:TP=-1.5:LRA=11" -c:a aac -b:a 192k -y "final_ready.mp4"
```

**EBU R128 Parameters:**
- `I=-16`: Integrated loudness target (-16 LUFS for broadcast)
- `TP=-1.5`: True peak limit (-1.5 dBFS)
- `LRA=11`: Loudness range (11 LU)

### Step 9: Verify Final Output

Use ffprobe to confirm the final video properties:

```bash
# JSON format for detailed analysis
ffprobe -v quiet -print_format json -show_format -show_streams "final_ready.mp4"

# Or quick check
ffprobe -v error -show_entries format=duration,bit_rate:stream=codec_name,width,height,channels,sample_rate -of default=noprint_wrappers=1 "final_ready.mp4"
```

**Verify:**
- ✓ Total duration matches expected (sum of audio durations)
- ✓ Audio is present and correctly mixed
- ✓ Video codec, resolution, fps preserved
- ✓ File size is reasonable

### Step 8: Cleanup Temporary Files

Remove all intermediate files created during processing, keeping only the final output:
*_video.mp4", "audio*.m4a", "final_merged.mp4", "concat_list.txt" -ErrorAction SilentlyContinue

# Option 2: Pattern-based cleanup
$tempFiles = @(
    "clip*_trimmed.*",
    "clip*_synced.*", 
    "clip*_final.*",
    "*_video.mp4",
    "audio*.m4a",
    "audio*.wav",
    "final_merged.*",
    "*concat*.txt"
)
$tempFiles | ForEach-Object { Remove-Item $_ -ErrorAction SilentlyContinue }

# Option 3: Interactive review before deletion
Get-ChildItem | Where-Object { 
    $_.Name -match "(trimmed|synced|final|_video|merged|concat)" -and
    $_.Name -notmatch "final_ready"
} | Format-Table Name, Length, LastWriteTime

# If confirmed, delete:
Get-ChildItem | Where-Object { 
    $_.Name -match "(trimmed|synced|final|_video
# If confirmed, delete:
Get-ChildItem | Where-Object { 
    $_.Name -match "(trimmed|synced|final|merged|concat)" -and
    $_.Name -notmatch "final_ready"
} | Remove-Item -Force
```

**Files to Keep:**
- ✓ `final_ready.mp4` (or your final output name)
- ✓ Original source files (never delete these!)

**Files to Delete:**
- `*_trimmed.*` - Extracted segments
- `*_synced.*` - Speed-adjusted clips
- `*_final.*` - Individual clips with audio (before merge)
- `*_video.mp4` - Converted images (e.g., image1_video.mp4)
- `final_merged.*` - Pre-normalization merged file
- `audio*.m4a`, `audio*.wav` - Extracted audio segments
- `*concat*.txt` - Concatenation list files

**Safety Tips:**
- Always verify final output before cleanup
- Use `-WhatIf` flag in PowerShell to preview deletions
- Keep backups of original source files
- Consider using a temporary working directory for intermediates

### Step 10: Output Format Conversion (If Requested)

If user requests a specific output format (audio-only, video-only, or different container format):

**Audio-Only Extraction:**
```bash
# Extract as MP3 (most compatible)
ffmpeg -i "final_ready.mp4" -vn -c:a libmp3lame -b:a 192k -y "final_audio.mp3"

# Extract as AAC/M4A (high quality)
ffmpeg -i "final_ready.mp4" -vn -c:a aac -b:a 192k -y "final_audio.m4a"

# Extract as WAV (uncompressed/lossless)
ffmpeg -i "final_ready.mp4" -vn -c:a pcm_s16le -y "final_audio.wav"

# Extract as FLAC (lossless compressed)
ffmpeg -i "final_ready.mp4" -vn -c:a flac -y "final_audio.flac"

# Extract as OGG Vorbis (open source)
ffmpeg -i "final_ready.mp4" -vn -c:a libvorbis -q:a 6 -y "final_audio.ogg"

# Extract as Opus (modern, efficient)
ffmpeg -i "final_ready.mp4" -vn -c:a libopus -b:a 128k -y "final_audio.opus"
```

**Video-Only Extraction (Silent):**
```bash
# Extract video without audio (stream copy - fastest)
ffmpeg -i "final_ready.mp4" -an -c:v copy -y "final_silent.mp4"

# Extract as WebM (VP9, web-optimized)
ffmpeg -i "final_ready.mp4" -an -c:v libvpx-vp9 -crf 30 -b:v 0 -y "final_silent.webm"

# Extract to MKV container
ffmpeg -i "final_ready.mp4" -an -c:v copy -y "final_silent.mkv"
```

**Full Format Conversion:**
```bash
# MP4 to WebM (full conversion)
ffmpeg -i "final_ready.mp4" -c:v libvpx-vp9 -crf 30 -b:v 0 -c:a libopus -b:a 128k -y "final_ready.webm"

# MP4 to MKV (container change, no re-encode)
ffmpeg -i "final_ready.mp4" -c copy -y "final_ready.mkv"

# Create GIF (from video)
ffmpeg -i "final_ready.mp4" -vf "fps=10,scale=480:-1:flags=lanczos" -y "final.gif"
```

**Key Parameters:**
- `-vn`: **Disable video** (audio-only output)
- `-an`: **Disable audio** (video-only output)
- `-c copy`: Stream copy (no re-encoding, fast)
- `-c:a CODEC`: Specify audio codec (libmp3lame, aac, flac, etc.)
- `-c:v CODEC`: Specify video codec (libx264, libvpx-vp9, etc.)
- `-b:a BITRATE`: Audio bitrate (192k recommended for MP3/AAC)
- `-q:a QUALITY`: Quality scale for OGG (0-10, 6 recommended)

**Audio Format Comparison:**
| Format | Codec | Quality | Size | Use Case |
|--------|-------|---------|------|----------|
| MP3 | libmp3lame | Good | Small | Universal compatibility |
| AAC/M4A | aac | Better | Small | Apple/mobile |
| WAV | pcm_s16le | Lossless | Very Large | Professional editing |
| FLAC | flac | Lossless | Large | Archival |
| OGG | libvorbis | Good | Small | Open source |
| Opus | libopus | Excellent | Very Small | Modern web |

**User Confirmation Required When:**
- User specifies output format (confirm format and quality settings)
- Converting to lossy format (warn about quality loss if applicable)
- Large file expected (estimate file size)

**For detailed format options, see:** [Output Formats Guide](./references/output-formats-guide.md)

## Tool Strategy

**Always prefer MCP tools first**, but immediately fall back to raw ffmpeg if:
- Tool returns timeout error
- Tool hangs without completing
- Tool returns unexpected results

**MCP tools to try first:**
- `mcp_ffmpeg-mcp_get_media_info`
- `mcp_ffmpeg-mcp_trim_video`
- `mcp_ffmpeg-mcp_change_speed`
- `mcp_ffmpeg-mcp_mix_audio`
- `mcp_ffmpeg-mcp_merge_videos`
- `mcp_ffmpeg-mcp_normalize_audio`
- `mcp_ffmpeg-mcp_run_custom_ffmpeg`

**Raw ffmpeg fallback commands:**
Use `run_in_terminal` with appropriate timeouts (60-180 seconds for most operations).

## Common Calculations

**Speed Factor for Video Stretching:**
```
factor = target_duration / current_duration

Examples:
- 15s → 32s: factor = 32/15 = 2.133 (slower)
- 22s → 29s: factor = 29/22 = 1.318 (slower)
- 30s → 15s: factor = 15/30 = 0.5 (faster)
```

**Timestamp Conversions:**
```
HH:MM:SS → seconds: (H*3600) + (M*60) + S
00:01:30 → 90 seconds

Seconds → HH:MM:SS: Use Windows calculator or:
90 Duration check: 37s video vs 61s audio → speed adjustment needed
3. No trimming needed (separate files)
4. Calculate splits: clip1→32s, clip2→29s (ratio: 32/15=2.13, 29/22=1.32)
5. Stretch: clip1→32s, clip2→29s (using setpts)
6. Extract audio segments: 0:38-1:10 (32s), 1:11-1:40 (29s)
7. Mix audio into stretched videos
8. Merge clips → final 61s video
9. (No duration control needed - matches target)
10. Normalize audio
11. Verify output
12medium | Balanced | Good | General use (default) |
| slow | Slower | Better | Final output |
| veryslow | Slowest | Best | Archive master |

| CRF | Quality | File Size |
|-----|---------|-----------|
| 18 | Near-lossless | Large |
| 23 | High (default) | Medium |
| 28 | Medium | Small |

## Duration check: 37s video vs 61s audio
3. Trim video segments using -ss/-t
4. Trim audio segments using -ss/-t
5. Continue with speed sync, mix, merge
6. (Duration control if needed)
7. Normalize
8. Verify output
9
**Issue:** Audio/video out of sync after speed change
- **Solution:** Use `-async 1` parameter or ensure `-an` when adjusting video speed

**Issue:** "Protocol not found" error
- **Solution:** Use absolute paths or escape backslashes in Windows paths

**Issue:** MCP tool hangs indefinitely
- **Solution:** Cancel (Ctrl+C) and immediately use raw ffmpeg command

**Issue:** Different resolutions/codecs between clips
- **Solution:** Re-encode all clips to same format before concat
Duration check and planning
3. Trim MainVideo segment
4. Extract voiceover segments for each clip
5. Sync speeds if needed
6. Mix audio
7. Merge: intro → main → outro
8. Duration control (if target specified)
9. Normalize
10. Verify
11
1. Get info: clip1=15s, clip2=22s, voiceover=61s
2. No trimming needed (separate files)
3. Calculate splits: clip1→32s, clip2→29s (ratio: 32/15=2.13, 29/22=1.32)
4. Stretch: clip1→32s, clip2→29s (using setpts)
5. Extract audio segments: 0:38-1:10 (32s), 1:11-1:40 (29s)
6. Mix audio into stretched videos
7. Merge clips → final 61s video
8. Normalize audio
9. Verify output
10. Cleanup temporary files
```

### Scenario B: Timestamped Segments from Large Files
```
Input: 
- BigVideo.mp4: Need 00:00:11-00:00:26 (15s) and 00:00:00-00:00:22 (22s)
- Audio.m4a: Need 00:00:38-00:01:10 (32s) and 00:01:11-00:01:40 (29s)

1. Get info on source files
2. Trim video segments using -ss/-t
3. Trim audio segments using -ss/-t
4. Continue with speed sync, mix, merge, normalize
5. Verify output
6. Cleanup: Remove trimmed/synced/final clips, keep final_ready.mp4
```

### Scenario C: Mixed Sources
```
Input:
- intro.mp4 (separate file)
- MainVideo.mp4 @ 00:05:00-00:06:30 (timestamped)
- outro.mp4 (separate file)
- voiceover.wav (single file for all)

1. Get info on all files
2. Trim MainVideo segment
3. Extract voiceover segments for each clip
4. Sync speeds if needed
5. Mix audio
6. Merge: intro → main → outro
7. Normalize
8. Verify
9. Cleanup: Remove extracted segments and intermediate files
```

### Scenario D: Images as Video Segments
```
Input:
- title.jpg → 00:00:00 to 00:00:05 (5 seconds)
- MainVideo.mp4 (full file, 30 seconds)
- credits.png → 00:00:35 to 00:00:40 (5 seconds)
- voiceover.wav (full, 40 seconds)

1. Get info on video and audio files
2. Convert images to video:
   - title.jpg → title_video.mp4 (5s duration, 1920x1080, 30fps)
   -image1_video.mp4`, `image2_video.mp4` - Converted images
3. `audio1.m4a`, `audio2.m4a` - Audio segments
4. `clip1_synced.mp4`, `clip2_synced.mp4` - After speed adjustment
5. `clip1_final.mp4`, `clip2_final.mp4` - After audio mixing
6. `final_merged.mp4` - After concatenation
7  - audio3: 35-40s for credits
4. Mix audio into each video segment
5. Merge: title → main → credits (40s total)
6. Normalize audio
7. Verify output
8. Cleanup: Remove title_video.mp4, credits_video.mp4, audio segments
```

## Integration Notes

- Works with existing Python pipeline scripts
- Can be invoked during or after script generation
- Handles outputs from video generation tools
- Compatible with manifest-based workflows

## File Naming Conventions

Recommended naming during workflow:
1. `clip1_trimmed.mp4`, `clip2_trimmed.mp4` - After extraction
2. `audio1.m4a`, `audio2.m4a` - Audio segments
3. `clip1_synced.mp4`, `clip2_synced.mp4` - After speed adjustment
4. `clip1_final.mp4`, `clip2_final.mp4` - After audio mixing
5. `final_merged.mp4` - After concatenation
6. `final_ready.mp4` - **FINAL OUTPUT** (keep this!)

This naming makes it easy to:
- Track progress through the workflow
- Debug issues at specific steps
- Identify temporary files for cleanup
- Distinguish final output from intermediates

**Cleanup:** All files except `final_ready.mp4` (and originals) can be safely deleted after verification.
