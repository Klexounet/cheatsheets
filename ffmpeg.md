## Get informations about video

### Duration
```
ffprobe -v error -show_entries format=duration -of default=noprint_wrappers=1:nokey=1 -i $INPUT
```

### Size
```
ffprobe -v error -select_streams v:0 -show_entries stream=width,height -of csv=p=0 -i $INPUT
```
[Source](https://superuser.com/questions/841235/how-do-i-use-ffmpeg-to-get-the-video-resolution?utm_medium=organic&utm_source=google_rich_qa&utm_campaign=google_rich_qa)

### Timecodes
```
ffmpeg -i $INPUT -filter:v 'showinfo' -f null /dev/null 2>&1  | grep -oE " n.*pos" | sed 's/n\://' | sed 's/pts\: *[0-9]* *pts_time\://' | sed 's/pos//' > $OUTPUT
```
où traditionnellement `OUTPUT=${INPUT%.*}.csv`

## Manipulate a video file

### Slice a video
```
ffmpeg -i $INPUT -t 01:23:45 -c copy $OUTPUT
```

### Extract (HD) frames from a video file
Option `-qscale` controls the quality of images, from range 2 (best quality) to 31 (worst quality)

Option `-filter:v fps=1` controls the fps to consider, if you want to extract one frame each second for example
```
ffmpeg -i $INPUT -qscale:v 2 $OUT_DIR/%07d.jpg
ffmpeg -i $INPUT -filter:v fps=1 -qscale:v 2 $OUT_DIR/%07d.jpg
```


### DeMultiplex
```
ffmpeg_options="-loglevel fatal -acodec copy -vcodec copy"
ffmpeg -f mpegts -i $ts_name \
           -map i:0x78  -map i:0x82  ${ffmpeg_options} ${slot_directory}m6.mp4 \
           -map i:0xdc  -map i:0xe6  ${ffmpeg_options} ${slot_directory}w9.mp4 \
           -map i:0x140 -map i:0x14a ${ffmpeg_options} ${slot_directory}arte.mp4 \
           -map i:0x1a4 -map i:0x1ae ${ffmpeg_options} ${slot_directory}france_5.mp4 \
           -map i:0x208 -map i:0x212 ${ffmpeg_options} ${slot_directory}6ter.mp4 \
           >> ${log_directory}/ffmpeg.log 2>&1
       ;;
```

### Restream to pipe / port
```
ffmpeg -re -i $INPUT -f mpegts pipe:1
ffmpeg -re -i $INPUT -f mpegts pipe:1 | nc $HOST $PORT
```
