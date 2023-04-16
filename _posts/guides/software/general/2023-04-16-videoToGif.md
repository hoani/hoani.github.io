---
title: "Convert Video To Gif"
excerpt: "Easy trick to convert videos to gif"
permalink: /guides/software/video-to-gif
categories:
  - guide
  - software
---

To convert video to GIF install `ffmpeg` and `gifsicle` then run:

```
ffmpeg -i input.mp4 -s 640x360 -pix_fmt rgb24 -r 20 -f gif - | gifsicle --optimize=3 --delay=3 > output.gif
```

Make sure your size parameter is the same ratio as your original file, or it will get squashed.

Example output:
![serial-pixel LEDs](/assets/images/posts/blog/chatbox/leds.gif)
