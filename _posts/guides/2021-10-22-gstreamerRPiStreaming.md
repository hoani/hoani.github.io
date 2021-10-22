---
title: "GStreamer RaspberryPi Streaming"
excerpt: "Streaming from a Raspberry Pi"
toc: true
categories:
  - guide
  - software
  - software-guide
---

Prerequisites:
* [GStreamer installed](https://hoani.net/posts/guides/2021-10-21-gstreamerSetup/)
* [Raspberry Pi Cam Setup](https://hoani.net/posts/guides/2021-09-09-raspberryPiCamera/)

## Create a stream on the Pi

We can use either of the following to setup a pipeline. I prefer the first one:

```sh
gst-launch-1.0 v4l2src device=/dev/video0 ! \
video/x-h264,width=640,height=480,framerate=30/1 ! \
h264parse ! \
rtph264pay config-interval=1 pt=96 ! \
gdppay ! \
tcpserversink host=192.168.50.202 port=5000
```

```sh
raspivid -t 0 -h 720 -w 1080 -fps 25 -hf -b 2000000 -o - | \
gst-launch-1.0 -v fdsrc ! \
h264parse ! \
rtph264pay config-interval=1 pt=96 ! \
gdppay ! \
tcpserversink host=192.168.50.202 port=5000
```

## Watch the stream using gstreamer

```sh
gst-launch-1.0 -v tcpclientsrc host=192.168.50.202 port=5000 ! \
gdpdepay ! \
rtph264depay ! \
avdec_h264 ! \
videoconvert ! \
autovideosink sync=false
```

`sync=false` means we always get the latest frame. This helps keep the latency as low as possible.

