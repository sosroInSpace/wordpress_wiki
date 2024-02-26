## Resize for web

```
ffmpeg -i "INPUT" -vcodec h264 -s 1280x720 -aspect 16:9 -r 30 -pix_fmt yuv420p -b:v 2200000 -profile:v main -acodec aac -b:a 96000 -ar 48000 OUTPUT
```

Guide:
- `-vcodec h264` - Video codec.
- `-s 1280x720 -aspect 16:9` - scale while maintaining aspect ratio (letterboxes video).
- `-r 30` - force 30 fps framerate.
- `-pix_fmt yuv420p` - allow quicktime compatibility.
- `-b:v 2200000` - bitrate of 2200KBps.
- `-profile:v main` - use `main` profile for video. Increases compatibility.
- `-acodec aac` - Audio codec.
- `-b:a 96000` - Audio bitrate.
- `-ar 48000` - Audio frequency (in Hz).

## Resize for web (no audio)

```
ffmpeg -i "INPUT" -vcodec h264 -s 1280x720 -aspect 16:9 -r 30 -pix_fmt yuv420p -b:v 2200000 -profile:v main -an OUTPUT
```

Guide:
- `-an` - no audio.

## Thumbnail video.

Based on MediaConvert HivoProdMP4Thumb (Time from 0s to 1s in this example).

```
ffmpeg -ss 0 -i "INPUT" -t 1 -vcodec h264 -vf scale=300:300:force_original_aspect_ratio=decrease,pad=300:300:0:140,setsar=1 -r 10 -pix_fmt yuv420p -b:v 128000 -profile:v main -an OUTPUT
```

## Thumbnail JPEG.

Based on MediaConvert HivoProdJPEGThumb.

`00:00:00.00` refers to `HOURS:MM:SS.MILLISECONDS`.

```
ffmpeg -i output.mp4 -ss 00:00:00.00 -vframes 1 out.jpg
```
