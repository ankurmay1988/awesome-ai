# Image-to-Video Conversion Guide

Complete reference for converting static images to video segments in your assembly workflow.

## Basic Conversion

```bash
# Simple conversion with duration
ffmpeg -loop 1 -i "image.jpg" -t 5 -c:v libx264 -pix_fmt yuv420p -r 30 -y "output.mp4"
```

**Parameters Explained:**
- `-loop 1`: Loop the single image (required for still images)
- `-i "image.jpg"`: Input image file
- `-t 5`: Duration in seconds (how long to display the image)
- `-c:v libx264`: Video codec (H.264)
- `-pix_fmt yuv420p`: Pixel format for compatibility
- `-r 30`: Frame rate (match your other video clips)
- `-y`: Overwrite output file if exists

## Resolution Matching

### Match Specific Resolution with Padding
```bash
# 1920x1080 with black letterbox/pillarbox
ffmpeg -loop 1 -i "image.jpg" -t 5 \
  -c:v libx264 -preset medium -crf 18 -pix_fmt yuv420p \
  -vf "scale=1920:1080:force_original_aspect_ratio=decrease,pad=1920:1080:(ow-iw)/2:(oh-ih)/2" \
  -r 30 -y "output.mp4"
```

### Custom Background Color
```bash
# White background instead of black
-vf "scale=1920:1080:force_original_aspect_ratio=decrease,pad=1920:1080:(ow-iw)/2:(oh-ih)/2:color=white"

# Custom RGB color (e.g., dark blue)
-vf "scale=1920:1080:force_original_aspect_ratio=decrease,pad=1920:1080:(ow-iw)/2:(oh-ih)/2:color=0x1a1a3e"
```

## Scale Filter Options

| Filter | Behavior | Use Case |
|--------|----------|----------|
| `scale=1920:1080` | Stretch to exact size | May distort aspect ratio |
| `scale=1920:-2` | Scale width, auto height | Maintain aspect, width priority |
| `scale=-2:1080` | Auto width, scale height | Maintain aspect, height priority |
| `scale=1920:1080:force_original_aspect_ratio=decrease` | Fit inside box | No distortion, may not fill |
| `scale=...decrease,pad=...` | Fit and letterbox | Perfect for matching video resolution |

**Note:** `-2` means auto-calculate to maintain aspect ratio (must be even number)

## Duration Calculation from Timestamps

If user provides timestamp range, calculate duration:

```
Start: 00:00:05  End: 00:00:12
Duration = End - Start = 12 - 5 = 7 seconds

Start: 00:01:30  End: 00:02:00
Duration = (1*60+30) - (2*60+0) = 90 - 120 = 30 seconds
```

## Quality Settings for Images

### High Quality (Production)
```bash
ffmpeg -loop 1 -i "image.png" -t 10 \
  -c:v libx264 -preset slow -crf 18 -pix_fmt yuv420p \
  -vf "scale=1920:1080:force_original_aspect_ratio=decrease,pad=1920:1080:(ow-iw)/2:(oh-ih)/2" \
  -r 30 -y "output.mp4"
```

### Medium Quality (Fast Preview)
```bash
ffmpeg -loop 1 -i "image.jpg" -t 10 \
  -c:v libx264 -preset fast -crf 23 -pix_fmt yuv420p \
  -r 30 -y "output.mp4"
```

### Maximum Quality (Archival)
```bash
ffmpeg -loop 1 -i "image.png" -t 10 \
  -c:v libx264 -preset veryslow -crf 15 -pix_fmt yuv420p \
  -vf "scale=1920:1080:force_original_aspect_ratio=decrease,pad=1920:1080:(ow-iw)/2:(oh-ih)/2" \
  -r 30 -y "output.mp4"
```

## Supported Image Formats

### Best Compatibility
- **JPEG/JPG** - Universal support, good for photos
- **PNG** - Supports transparency, good for graphics

### Also Supported
- BMP (large files)
- TIFF (may need specific ffmpeg build)
- WebP (modern format, check ffmpeg build)
- GIF (single frame extracted)

### Transparency Handling
PNG images with transparency will have black background by default. Use custom color:
```bash
-vf "scale=...,pad=...:color=white"
```

## Common Use Cases

### Title Card (5 seconds)
```bash
ffmpeg -loop 1 -i "title.jpg" -t 5 \
  -c:v libx264 -preset medium -crf 18 -pix_fmt yuv420p \
  -vf "scale=1920:1080:force_original_aspect_ratio=decrease,pad=1920:1080:(ow-iw)/2:(oh-ih)/2" \
  -r 30 -y "title_video.mp4"
```

### End Credits (10 seconds)
```bash
ffmpeg -loop 1 -i "credits.png" -t 10 \
  -c:v libx264 -preset medium -crf 18 -pix_fmt yuv420p \
  -vf "scale=1920:1080:force_original_aspect_ratio=decrease,pad=1920:1080:(ow-iw)/2:(oh-ih)/2" \
  -r 30 -y "credits_video.mp4"
```

### Photo Slideshow
```bash
# Convert each photo (different durations)
ffmpeg -loop 1 -i "photo1.jpg" -t 3 -c:v libx264 -pix_fmt yuv420p -r 30 -y "photo1.mp4"
ffmpeg -loop 1 -i "photo2.jpg" -t 5 -c:v libx264 -pix_fmt yuv420p -r 30 -y "photo2.mp4"
ffmpeg -loop 1 -i "photo3.jpg" -t 4 -c:v libx264 -pix_fmt yuv420p -r 30 -y "photo3.mp4"

# Then merge using concat (see main workflow)
```

## Combining Images with Video Clips

### Workflow Order
1. Convert images to video segments with calculated durations
2. Process video clips (trim, speed adjust if needed)
3. Mix audio into both image-videos and regular videos
4. Concatenate all segments in correct order
5. Normalize and finalize

### Example Sequence
```
title_video.mp4 (5s) → clip1_final.mp4 (32s) → clip2_final.mp4 (29s) → credits_video.mp4 (5s)
Total: 71 seconds
```

## Troubleshooting

### "Invalid pixel format"
**Problem:** Image conversion fails  
**Solution:** Always include `-pix_fmt yuv420p`

### Black bars appear around image
**Problem:** Aspect ratio mismatch  
**Solution:** This is expected (letterbox/pillarbox) when using `pad` filter. To avoid, use `scale=1920:1080` but this may distort.

### Image looks pixelated/low quality
**Problem:** Too much compression or low source quality  
**Solution:** 
- Use PNG source if available
- Increase quality: `-crf 18` or lower
- Use slower preset: `-preset slow`
- Ensure source image is high resolution

### File size too large
**Problem:** Image-video creates huge file  
**Solution:** 
- Lower CRF: `-crf 23` or higher (18 is very high quality)
- Faster preset: `-preset fast`
- Consider source image resolution

### Wrong frame rate causes stutter in final video
**Problem:** Image video has different fps than other clips  
**Solution:** Always match frame rate with `-r 30` (or whatever your other clips use)

## Frame Rate Matching

**Critical:** All segments (images and videos) must have the same frame rate!

```bash
# Check frame rate of existing video
ffprobe -v error -select_streams v:0 -show_entries stream=r_frame_rate -of default=noprint_wrappers=1:nokey=1 "video.mp4"

# Use that frame rate for image conversion
ffmpeg -loop 1 -i "image.jpg" -t 5 -r 30 ...
```

## Best Practices

1. **Match resolution**: Use same resolution as your video clips
2. **Match frame rate**: Use `-r` to match video clip fps
3. **Use padding**: Better than stretching which distorts
4. **High quality sources**: Start with high-resolution images
5. **Consistent naming**: Use `*_video.mp4` suffix for converted images
6. **Clean up**: Remove `*_video.mp4` files after final merge

## Quick Reference Commands

```bash
# Standard 1920x1080, 30fps, 5 seconds
ffmpeg -loop 1 -i "IMAGE.jpg" -t 5 -c:v libx264 -preset medium -crf 18 -pix_fmt yuv420p -vf "scale=1920:1080:force_original_aspect_ratio=decrease,pad=1920:1080:(ow-iw)/2:(oh-ih)/2" -r 30 -y "output.mp4"

# Fast preview (lower quality)
ffmpeg -loop 1 -i "IMAGE.jpg" -t 5 -c:v libx264 -preset fast -crf 23 -pix_fmt yuv420p -r 30 -y "output.mp4"

# Match existing video properties
RESOLUTION=$(ffprobe -v error -select_streams v:0 -show_entries stream=width,height -of csv=s=x:p=0 video.mp4)
FPS=$(ffprobe -v error -select_streams v:0 -show_entries stream=r_frame_rate -of default=noprint_wrappers=1:nokey=1 video.mp4 | bc)
ffmpeg -loop 1 -i "IMAGE.jpg" -t 5 -c:v libx264 -pix_fmt yuv420p -vf "scale=$RESOLUTION:force_original_aspect_ratio=decrease,pad=$RESOLUTION:(ow-iw)/2:(oh-ih)/2" -r $FPS -y "output.mp4"
```
