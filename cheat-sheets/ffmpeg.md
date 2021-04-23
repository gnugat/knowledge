# ffmpeg cheat sheet

A highly opinionated ffmpeg cheat sheet.

## Slow down / Speed up video & audio

* video: `setpts=<speed>*PTS`, with `0.5` to speed up x2, `0.25` for x4, etc
* audio: `atempo=<speed>`, with `2.0` to speed up x2
  * ⚠️ limited between `0.5` and `2.0`, to speed up x4 string 2 of them (`atempo=2.0,atempo=2.0`)

```
ffmpeg -i ./input.mp4 -filter_complex "[0:v]setpts=0.25*PTS[v];[0:a]atempo=2.0,atempo=2.0[a]" -map "[v]" -map "[a]" output.mp4
```

[source](https://trac.ffmpeg.org/wiki/How%20to%20speed%20up%20/%20slow%20down%20a%20video)
