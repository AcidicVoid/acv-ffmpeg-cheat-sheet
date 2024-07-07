# AcidicVoid's FFmpeg cheat sheet
## acv-ffmpeg-cheat-sheet

Most of the prompts use the h264_nvenc video codec.
For alternatives, see:  
https://ffmpeg.org/ffmpeg-codecs.html#Video-Encoders

### Compress large MKV files (to mp4)
Very simple!
```
ffmpeg -i input.mkv -map 0 -vcodec h264_nvenc -crf 22 -vf yadif -c:a ac3 -c:s copy output.mp4
```

### Combining audio and video from different sources
video from input_a and audio from input_b  
*(aac audio codec with low bitrate for experimental purposes)*
```
ffmpeg -i input_a.mp4 -i input_b.mp4 -c:v copy -c:a aac -b:a 48k -map 0:v:0 -map 1:a:0 output.mp4
```
video from input_a and audio from input_b, better method with cuda
```
for %i in (*.mkv) do ffmpeg -hwaccel cuda -hwaccel_output_format cuda -threads 16 -i "%i" -map 0 -preset slow -vcodec h264_nvenc -crf 23 -c:a ac3 -c:s copy "%~ni.mkv"

### Convert mp4 to V9 webm
Very simple!
```
ffmpeg -i input.mp4 -c:v libvpx-vp9 -b:v 2M output.webm
```

### Selecting subtitles by language
Advanced example to select specific audio and subtitle streams before compressing a MKV file. The Streams are selected by language!
```
ffmpeg -i input.mkv -map 0 -map 0:a:m:language:ger -map 0:a:m:language:jpn -map 0:s:m:language:ger -vcodec h264_nvenc -crf 16 -vf yadif -c:a ac3 -c:s mov_text output.mp4
```

### Converting a VOB file, using PAL aspect ratio
Oddly specific, isn't it?
```
ffmpeg -i input.vob -vf "scale=768:576,setsar=1" -c:v h264_nvenc -preset slow -c:a aac -b:a 256k output.mp4
```

### Slowdown
Meant for slowing down videos with very high framerates like outputs from frame interpolation tool.  
This prompt assumes the input file comes with 240fps. We'll get a four times slower video with 60fps, without re-encoding.
```
ffmpeg -y -i input.mp4 -vf "setpts=4.0*PTS" -r 60 output.mp4
```

Slowdown via full CUDA hardware transcode with NVDEC and NVENC.  
See: https://trac.ffmpeg.org/wiki/HWAccelIntro#CUDANVENCNVDEC
```
ffmpeg -hwaccel cuda -hwaccel_output_format cuda -threads 8 -i input.mp4 -c:v h264_nvenc -preset slow -vf "setpts=4.0*PTS" -r 60 output.mp4
```

### Decimate frames
See: https://stackoverflow.com/questions/37088517/remove-sequentially-duplicate-frames-when-using-ffmpeg  
Uses CUDA in this case (not mandatory): https://docs.nvidia.com/video-technologies/video-codec-sdk/12.0/ffmpeg-with-nvidia-gpu/index.html
```
ffmpeg -hwaccel cuda -threads 8 -i input.mp4 -vf decimate=mixed=true output.mp4
```

### Change the framerate only
Very simple!
```
ffmpeg -i input.mp4 -filter:v fps=fps=60 -fps_mode vfr output.mp4
```

### WAV to mp3
Very simple!
```
ffmpeg -i input.wav -vn -ar 48000 -ac 2 -b:a 320k output.mp3
```
