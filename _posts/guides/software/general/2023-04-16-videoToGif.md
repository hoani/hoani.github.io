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

For pixel art videos, we may want to generate our own pallete. using the `neighbor` flag also helps scale the pixels with less blur.

```
ffmpeg -i input.mp4 -filter_complex "fps=15, scale=-1:360:flags=neighbor[s]; [s]split[a][b]; [a]palettegen[palette]; [b][palette]paletteuse" -f gif - | gifsicle --optimize=5 --delay=3 > output.gif
```

Example output:
![clover gif](/assets/images/posts/blog/clover-trajectory/intercept.gif)
