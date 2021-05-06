# ffmpeg cheat sheet

A highly opinionated ffmpeg cheat sheet.

* find video FPS: `ffmpeg -i ./input 2>&1 | sed -n "s/.*, \(.*\) fp.*/\1/p"`
* slow down video x2, remove audio: `ffmpeg -i ./input -r 15 -filter:v "setpts=2.0*PTS" -an ./output`
* speed up video x2, remove audio: `ffmpeg -i ./input -r 60 -filter:v "setpts=0.5*PTS" -an ./output`
* speed up video & audio x2: `ffmpeg -i ./input -r 60 -filter_complex "[0:v]setpts=0.5*PTS[v];[0:a]atempo=2.0[a]" -map "[v]" -map "[a]" ./output`
* speed up video & audio x4: `ffmpeg -i ./input -r 120 -filter_complex "[0:v]setpts=0.25*PTS[v];[0:a]atempo=2.0,atempo=2.0[a]" -map "[v]" -map "[a]" ./output`
* add audio to video: `ffmpeg -i ./video -i ./audio -filter_complex amix=inputs=2:duration=shortest ./output`
* extract last frame from video: `ffmpeg -sseof -3 -i input -update 1 -q:v 1 last.jpg`

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
