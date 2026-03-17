# Duration Management Guide

Complete reference for managing video duration when assembling content that doesn't match the target length.

## Overview

When assembling video from multiple clips, you may have:
- **Content longer than target** → Need to compress, speed up, or trim
- **Content shorter than target** → Need to extend, slow down, or accept shorter
- **Content matches target** → Perfect, proceed normally

This guide covers all strategies with quality impact analysis and user decision points.

## Duration Calculation

### Calculate Total Content Duration

```powershell
# From ffprobe results
$clip1Duration = 15.005  # seconds
$clip2Duration = 22.016
$image1Duration = 5
$totalDuration = $clip1Duration + $clip2Duration + $image1Duration
# Total: 42.021 seconds

# Compare to target
$targetDuration = 60  # seconds
$difference = $targetDuration - $totalDuration
# Difference: +17.979 seconds (too short)

# Calculate speed factor if adjusting
$speedFactor = $targetDuration / $totalDuration
# Factor: 1.428x (need to slow down)
```

### Quality Threshold Check

```powershell
function Test-QualityThreshold {
    param(
        [double]$speedFactor,
        [double]$maxSpeedWarn = 1.5,
        [double]$maxSpeedError = 2.5
    )
    
    if ($speedFactor -gt $maxSpeedError -or $speedFactor -lt (1/$maxSpeedError)) {
        Write-Host "🛑 CRITICAL: Speed change ${speedFactor}x is extreme!" -ForegroundColor Red
        Write-Host "   This will be very obvious to viewers." -ForegroundColor Red
        return "CRITICAL"
    }
    elseif ($speedFactor -gt $maxSpeedWarn -or $speedFactor -lt (1/$maxSpeedWarn)) {
        Write-Host "⚠️ WARNING: Speed change ${speedFactor}x may be noticeable" -ForegroundColor Yellow
        return "WARNING"
    }
    else {
        Write-Host "✓ Speed change ${speedFactor}x is acceptable" -ForegroundColor Green
        return "OK"
    }
}

# Usage
$status = Test-QualityThreshold -speedFactor 1.428
```

## Strategies for LONGER Content (Content > Target)

### Strategy 1: Speed Up Proportionally ⚡

**When to Use:**
- Speed factor < 1.5x (acceptable quality)
- All content is important
- Audio quality is not critical

**Quality Impact:**
- 1.0-1.3x: Barely noticeable ✓
- 1.3-1.5x: Slightly noticeable, usually acceptable ⚠️
- 1.5-2.0x: Noticeably faster, chipmunk voices ⚠️⚠️
- 2.0x+: Very obvious, poor quality 🛑

**Implementation:**

```bash
# Speed up entire merged video (video + audio)
ffmpeg -i "merged.mp4" \
  -filter_complex "[0:v]setpts=PTS/SPEED[v];[0:a]atempo=SPEED[a]" \
  -map "[v]" -map "[a]" \
  -c:v libx264 -preset medium -crf 18 -c:a aac \
  -y "adjusted.mp4"

# Example: 70s → 60s (speed = 70/60 = 1.167x)
ffmpeg -i "merged.mp4" \
  -filter_complex "[0:v]setpts=PTS/1.167[v];[0:a]atempo=1.167[a]" \
  -map "[v]" -map "[a]" \
  -c:v libx264 -preset medium -crf 18 -c:a aac \
  -y "adjusted.mp4"
```

**atempo Filter Limits:**
- Range: 0.5x to 2.0x per filter
- For >2x: Chain multiple atempo filters
- Example: 2.5x = atempo=2.0,atempo=1.25

### Strategy 2: Trim End Content ✂️

**When to Use:**
- Speed change would be > 1.5x
- End content is less important
- User can decide what to cut

**Quality Impact:** None (no re-encoding if using copy)

**Implementation:**

```bash
# Simply trim to target duration (keep beginning)
ffmpeg -i "merged.mp4" -t TARGET_DURATION -c copy -y "trimmed.mp4"

# Example: 70s video → 60s (cut last 10s)
ffmpeg -i "merged.mp4" -t 60 -c copy -y "trimmed.mp4"

# Keep specific segment (trim from both ends)
ffmpeg -ss START_TIME -i "merged.mp4" -t DURATION -c copy -y "trimmed.mp4"
```

### Strategy 3: Selective Trimming 🎯

**When to Use:**
- User can identify specific clips to shorten/remove
- Want maximum control over content
- Different clips have different priorities

**Quality Impact:** None for removed content, depends on adjustment for kept content

**Implementation:**

```powershell
# User decision tree
Write-Host "Content duration: 70s, Target: 60s (10s too long)"
Write-Host ""
Write-Host "Options:"
Write-Host "1. Remove title card (5s) → New total: 65s (still 5s over)"
Write-Host "2. Remove outro (7s) → New total: 63s (still 3s over)"
Write-Host "3. Trim clip2 by 5s → New total: 65s"
Write-Host "4. Speed up everything by 1.167x"
Write-Host "5. Combination: Remove outro (7s) + speed up 1.05x"

# After user selects, rebuild concat list without removed clips
# And/or adjust individual clip durations
```

### Strategy 4: Smart Compression (File Size) 💾

**When to Use:**
- Target is file size limit, not duration
- Video platforms with size restrictions

**Quality Impact:** Medium to High (visible artifacts if CRF > 28)

**Implementation:**

```bash
# Two-pass encoding for target bitrate
ffmpeg -i "input.mp4" -c:v libx264 -preset slow -b:v TARGET_BITRATE -pass 1 -f null -
ffmpeg -i "input.mp4" -c:v libx264 -preset slow -b:v TARGET_BITRATE -pass 2 -c:a aac -b:a 128k -y "output.mp4"

# Calculate target bitrate for file size
# Bitrate (kbps) = (Target Size MB * 8192) / Duration (seconds)
# Example: 50MB in 60s = (50 * 8192) / 60 = 6827 kbps
```

## Strategies for SHORTER Content (Content < Target)

### Strategy 1: Slow Down Proportionally 🐌

**When to Use:**
- Speed factor < 2.0x (acceptable quality)
- Want to fill entire duration
- Slow-motion effect is acceptable

**Quality Impact:**
- 1.0-1.5x: Slightly slower, usually fine ✓
- 1.5-2.0x: Slow motion, may be dramatic ⚠️
- 2.0x+: Very slow, may look unnatural 🛑

**Implementation:**

```bash
# Slow down entire video
ffmpeg -i "merged.mp4" \
  -filter_complex "[0:v]setpts=PTS*FACTOR[v];[0:a]atempo=1/FACTOR[a]" \
  -map "[v]" -map "[a]" \
  -c:v libx264 -preset medium -crf 18 -c:a aac \
  -y "slowed.mp4"

# Example: 40s → 60s (factor = 60/40 = 1.5x slower)
# Video: setpts=PTS*1.5
# Audio: atempo=1/1.5 = atempo=0.667
ffmpeg -i "merged.mp4" \
  -filter_complex "[0:v]setpts=PTS*1.5[v];[0:a]atempo=0.667[a]" \
  -map "[v]" -map "[a]" \
  -c:v libx264 -preset medium -crf 18 -c:a aac \
  -y "slowed.mp4"
```

### Strategy 2: Add Black Padding ⬛

**When to Use:**
- Don't want to alter content speed
- Fade to black is acceptable
- Documentary/artistic style

**Quality Impact:** None

**Implementation:**

```bash
# Add black frames at end
ffmpeg -i "merged.mp4" \
  -vf "tpad=stop_mode=add:stop_duration=EXTRA_SECONDS" \
  -af "apad=pad_dur=EXTRA_SECONDS" \
  -y "padded.mp4"

# Example: 40s → 60s (add 20s black)
ffmpeg -i "merged.mp4" \
  -vf "tpad=stop_mode=add:stop_duration=20" \
  -af "apad=pad_dur=20" \
  -y "padded.mp4"

# Clone last frame instead of black
ffmpeg -i "merged.mp4" \
  -vf "tpad=stop_mode=clone:stop_duration=20" \
  -af "apad=pad_dur=20" \
  -y "padded.mp4"
```

### Strategy 3: Fade to Black + Extend 🌑

**When to Use:**
- More polished than abrupt black
- Professional look
- End credits style

**Quality Impact:** None

**Implementation:**

```bash
# Fade out last 2 seconds, then extend with black
CURRENT_DURATION=40
FADE_START=$((CURRENT_DURATION - 2))
EXTRA_SECONDS=$((60 - CURRENT_DURATION))

ffmpeg -i "merged.mp4" \
  -filter_complex "
    [0:v]fade=t=out:st=${FADE_START}:d=2,tpad=stop_mode=add:stop_duration=${EXTRA_SECONDS}[v];
    [0:a]afade=t=out:st=${FADE_START}:d=2,apad=pad_dur=${EXTRA_SECONDS}[a]
  " \
  -map "[v]" -map "[a]" \
  -y "extended.mp4"
```

### Strategy 4: Accept Shorter Duration ✓

**When to Use:**
- Target is not a hard requirement
- Quality more important than exact duration
- Platform allows flexible duration

**Quality Impact:** None

**Implementation:**
- No action needed
- Final video is shorter than target
- Document actual duration for user

## Decision Flow Chart

```
START: Calculate content duration vs target

Is content duration within ±5% of target?
├─ YES → Perfect! Proceed normally
└─ NO → Continue

Is content LONGER than target?
├─ YES → Content too long
│   │
│   └─→ Calculate speed factor = target/content
│       │
│       ├─ Factor < 1.5x?
│       │  ├─ YES → Speed up (acceptable quality)
│       │  └─ NO → Offer options:
│       │         1. Trim end content
│       │         2. Selective clip removal
│       │         3. Speed up anyway (warn user)
│       │         4. Ask user to choose
│
└─ NO → Content too short
    │
    └─→ Calculate slow factor = target/content
        │
        ├─ Factor < 2.0x?
        │  ├─ YES → Slow down (acceptable)
        │  └─ NO → Offer options:
        │         1. Add padding/black frames
        │         2. Fade to black + extend
        │         3. Slow down anyway (warn user)
        │         4. Accept shorter duration
```

## User Confirmation Prompts

### When to Prompt User

```powershell
$requireConfirmation = $false

# Check speed factor
if ($speedFactor -gt 1.5 -or $speedFactor -lt 0.67) {
    Write-Host "⚠️ Speed adjustment of ${speedFactor}x may affect quality"
    $requireConfirmation = $true
}

# Check if trimming significant content
$trimPercent = [Math]::Abs($difference) / $totalDuration * 100
if ($trimPercent -gt 10) {
    Write-Host "⚠️ Would trim ${trimPercent}% of content"
    $requireConfirmation = $true
}

# Check re-encoding count
if ($reencodeCount -gt 1) {
    Write-Host "⚠️ This would be re-encode #${reencodeCount} (quality loss)"
    $requireConfirmation = $true
}

if ($requireConfirmation) {
    Write-Host ""
    Write-Host "Options:"
    Write-Host "1. Proceed anyway"
    Write-Host "2. Use alternative strategy"
    Write-Host "3. Cancel and adjust inputs"
    $choice = Read-Host "Select option (1-3)"
}
```

### Example Confirmation Messages

```
📊 Duration Analysis
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
Total content:  70.5 seconds
Target duration: 60.0 seconds
Difference:     +10.5 seconds (17.7% too long)

⚠️ Speed adjustment needed: 1.175x faster

Quality Impact Assessment:
✓ Speed change is within acceptable range (<1.5x)
✓ Voices will sound slightly faster but natural
✓ Visual motion slightly accelerated

Proceed with speed adjustment? (Y/n)
```

```
📊 Duration Analysis
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
Total content:  45.0 seconds
Target duration: 90.0 seconds
Difference:     -45.0 seconds (50% too short)

🛑 Slow-down factor: 2.0x slower (MAXIMUM RECOMMENDED)

Quality Impact Assessment:
⚠️ Content will be in slow motion
⚠️ May look unnatural for talking heads
✓ Could work for B-roll/action footage

Options:
1. Slow down by 2.0x (use all content, slow motion)
2. Add 45s black padding (keep normal speed)
3. Accept 45s video (shorter than target)
4. Fade out and extend with black

Select option (1-4):
```

## Best Practices

### DO:
✓ Always calculate and show user the speed factor  
✓ Warn when speed > 1.5x or < 0.67x  
✓ Offer multiple options when quality threshold exceeded  
✓ Consider content type (talking heads vs B-roll)  
✓ Test short preview before full encode  
✓ Document actual final duration

### DON'T:
✗ Apply extreme speed changes (>2.5x) without warning  
✗ Trim content without user knowing what's cut  
✗ Re-encode multiple times unnecessarily  
✗ Use lossy compression for duration control  
✗ Assume user wants exact duration over quality

## Quality Threshold Reference

| Factor | Video | Audio | Recommendation |
|--------|-------|-------|----------------|
| 0.5x | Very slow | Very low pitch | 🛑 Avoid |
| 0.67x | Slow motion | Low pitch | ⚠️ Warn user |
| 0.8x | Slightly slow | Slightly low | ✓ Usually OK |
| 1.0x | Normal | Normal | ✓ Perfect |
| 1.2x | Slightly fast | Slightly high | ✓ Usually OK |
| 1.5x | Noticeably fast | High pitch | ⚠️ Warn user |
| 2.0x | Very fast | Very high pitch | 🛑 Avoid |
| 2.5x+ | Extremely fast | Chipmunk | 🛑 Never |

## Integration with Main Workflow

1. **After Step 1 (Analyze)**: Calculate total duration
2. **At Step 2**: Compare with target, show user options if mismatch
3. **Before Step 4 (Sync)**: Apply proportional speed if accepted
4. **After Step 6 (Merge)**: Apply trim/extend if needed
5. **Before Step 9 (Verify)**: Confirm final duration matches expectation

## Example PowerShell Helper Script

```powershell
function Get-DurationStrategy {
    param(
        [double]$ContentDuration,
        [double]$TargetDuration,
        [double]$MaxSpeedWarn = 1.5,
        [double]$MaxSpeedError = 2.5
    )
    
    $difference = $TargetDuration - $ContentDuration
    $percentDiff = [Math]::Abs($difference) / $ContentDuration * 100
    
    # If within 5%, no action needed
    if ($percentDiff -lt 5) {
        return @{
            Action = "None"
            Message = "Content duration matches target (within 5%)"
            SpeedFactor = 1.0
        }
    }
    
    $speedFactor = $TargetDuration / $ContentDuration
    
    $result = @{
        ContentDuration = $ContentDuration
        TargetDuration = $TargetDuration
        Difference = $difference
        PercentDiff = $percentDiff
        SpeedFactor = $speedFactor
    }
    
    # Determine recommendation
    if ($ContentDuration -gt $TargetDuration) {
        # Too long
        if ($speedFactor -lt (1 / $maxSpeedError)) {
            $result.Action = "TrimOrSelectiveRemoval"
            $result.Message = "🛑 Speed-up would be extreme (${speedFactor}x). Recommend trimming content."
            $result.Quality = "CRITICAL"
        }
        elseif ($speedFactor -lt (1 / $maxSpeedWarn)) {
            $result.Action = "SpeedUpWithWarning"
            $result.Message = "⚠️ Speed-up of ${speedFactor}x may be noticeable. Options: speed up or trim?"
            $result.Quality = "WARNING"
        }
        else {
            $result.Action = "SpeedUp"
            $result.Message = "✓ Speed-up of ${speedFactor}x is acceptable."
            $result.Quality = "OK"
        }
    }
    else {
        # Too short
        if ($speedFactor -gt $maxSpeedError) {
            $result.Action = "PaddingOrAcceptShorter"
            $result.Message = "🛑 Slow-down would be extreme (${speedFactor}x). Recommend padding or accepting shorter video."
            $result.Quality = "CRITICAL"
        }
        elseif ($speedFactor -gt $maxSpeedWarn) {
            $result.Action = "SlowDownWithWarning"
            $result.Message = "⚠️ Slow-down of ${speedFactor}x may create slow-motion effect. Options: slow down or add padding?"
            $result.Quality = "WARNING"
        }
        else {
            $result.Action = "SlowDown"
            $result.Message = "✓ Slow-down of ${speedFactor}x is acceptable."
            $result.Quality = "OK"
        }
    }
    
    return $result
}

# Usage example
$strategy = Get-DurationStrategy -ContentDuration 70.5 -TargetDuration 60
Write-Host $strategy.Message
Write-Host "Recommended action: $($strategy.Action)"
```
