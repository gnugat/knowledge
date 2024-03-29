# ffmpeg cheat sheet

A highly opinionated ffmpeg cheat sheet.

* find video FPS: `ffmpeg -i ./input 2>&1 | sed -n "s/.*, \(.*\) fp.*/\1/p"`
* slow down video x2, remove audio: `ffmpeg -i ./input -r 15 -filter:v "setpts=2.0*PTS" -an ./output`
* speed up video x2, remove audio: `ffmpeg -i ./input -r 60 -filter:v "setpts=0.5*PTS" -an ./output`
* speed up video & audio x2: `ffmpeg -i ./input -r 60 -filter_complex "[0:v]setpts=0.5*PTS[v];[0:a]atempo=2.0[a]" -map "[v]" -map "[a]" ./output`
* speed up video & audio x4: `ffmpeg -i ./input -r 120 -filter_complex "[0:v]setpts=0.25*PTS[v];[0:a]atempo=2.0,atempo=2.0[a]" -map "[v]" -map "[a]" ./output`
* add audio to video: `ffmpeg -i ./video -i ./audio -filter_complex amix=inputs=2:duration=shortest ./output`
* extract part of video (starting at 00:10, for 30s): `ffmpeg -ss 10 -i ./video -t 30 ./extract`
* extract audio from video: `ffmpeg -i ./video -q:a 0 -map a ./sound`
* extract first frame from video: `ffmpeg -i ./video -vframes 1 ./first.jpg`
* extract last frame from video: `ffmpeg -sseof -3 -i ./input -update 1 -q:v 1 last.jpg`
* add image at the beginning of video: `ffmpeg -loop 1 -framerate 30 -t 1 -i ./image -t 1 -f lavfi -i aevalsrc=0 -i ./video -filter_complex '[0:0] [1:0] [2:0] [2:1] concat=n=2:v=1:a=1' ./output`
* add image at the end of video: `ffmpeg -i ./video -loop 1 -t 1 -i ./image -f lavfi -t 1 -i anullsrc -filter_complex "[0:v] [0:a] [1:v] [2:a] concat=n=2:v=1:a=1 [v] [a]" -map "[v]" -map "[a]" ./output`

## Rotating videos

* strip rotation metadata: `ffmpeg -y -i ./video -c copy -metadata:s:v:0 rotate=0 ./output`
* rotate 90°: `ffmpeg -noautorotate -i ./video -vf 'rotate=90*(PI/180):bilinear=0' -metadata:s:v:0 rotate=0 -pix_fmt yuv420p -codec:v libx264 -codec:a copy ./output`

## Adding audio to video

* replace `0.mp4`'s audio with `1.mp4`'s (will contain one audio track):
  `ffmpeg -i 0.mp4 -i 1.mp4 -map 0:v -map 1:a -c copy -shortest output.mp4`
* add `1.mp4`'s audio to `0.mp4` (will contain two audio tracks):
  `ffmpeg -i 0.mp4 -i 1.mp4 -map 0 -map 1:a -c copy -shortest output.mp4`
* merge `1.mp4`'s audio into `0.mp4`'s (will contain one audio track):
  `ffmpeg -i 0.mp4 -i 1.mp4 -filter_complex "[0:a][1:a]amerge=inputs=2[a]" -map 0:v -map "[a]" -c copy -ac 2 -shortest output.mp4`

* `-map` allows to manually select streams / tracks ([ffmpeg map](https://trac.ffmpeg.org/wiki/Map))
* `-c copy` copies stream to avoid re-encoding (quality preserved, fast to process)
  * if input audio stream isn't compatible with output's, only copy video's stream using `-c:v copy`
* `-shortest` makes the output's duration the same as the shortest input's

[source](https://stackoverflow.com/a/11783474/3437428)

## Concatenating videos

To losslessly concatenate two mp4 videos, convert them to MPEG-2 transport streams:

```
ffmpeg -i input1.mp4 -c copy -bsf:v h264_mp4toannexb -f mpegts intermediate1.ts
ffmpeg -i input2.mp4 -c copy -bsf:v h264_mp4toannexb -f mpegts intermediate2.ts
ffmpeg -i "concat:intermediate1.ts|intermediate2.ts" -c copy -bsf:a aac_adtstoasc output.mp4
```

[source](https://trac.ffmpeg.org/wiki/Concatenate)

## Slow down / Speed up video & audio

* video: `setpts=<speed>*PTS`, with `0.5` to speed up x2, `0.25` for x4, etc
  * ⚠️ will drop FPS. To keep FPS, find original FPS and multiply it with option `-r <fps>
* audio: `atempo=<speed>`, with `2.0` to speed up x2
  * ⚠️ limited between `0.5` and `2.0`, to speed up x4 string 2 of them (`atempo=2.0,atempo=2.0`)

[source](https://trac.ffmpeg.org/wiki/How%20to%20speed%20up%20/%20slow%20down%20a%20video)
