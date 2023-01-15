---
title: "GStreamer RaspberryPi Streaming"
excerpt: "Streaming from a Raspberry Pi"
permalink: /guides/software/raspberrypi/gstreamer
toc: true
categories:
  - guide
  - software
  - software-guide
---

Prerequisites:
* [GStreamer installed](https://hoani.net/posts/guides/2021-10-21-gstreamerSetup/)
* [Raspberry Pi Cam Setup](https://hoani.net/posts/guides/2021-09-09-raspberryPiCamera/)

## Install rpicamsrc

`rpicamsrc` is officially part of gstreamer 1.18. However, at the time of this writing, the latest supported gstreamer on Raspberry Pi OS is 1.14. 

We can build and install `rpicamsrc` from source.

Firstly, install build dependencies:

```
sudo apt-get install autoconf automake libtool pkg-config \
libgstreamer1.0-dev libgstreamer-plugins-base1.0-dev \
libraspberrypi-dev
```

Now, clone and install the library from source:

```
git clone https://github.com/thaytan/gst-rpicamsrc
cd gst-rpicamsrc
./autogen.sh --prefix=/usr --libdir=/usr/lib/arm-linux-gnueabihf/
make
sudo make install
```

## Create a stream on the Pi

I like using `rpicamsrc` because it gives me really great control over the camera.

```sh
gst-launch-1.0 rpicamsrc \
sensor-mode=4 keyframe-interval=1 preview-opacity=50 ! \
h264parse ! \
rtph264pay config-interval=1 pt=96 ! \
gdppay ! \
tcpserversink host=0.0.0.0 port=5000
```

* `sensor-mode=4` gives the best viewing angle
* `keyframe-interval=1` sends each frame as a keyframe
  * This prevents streamed frames from going grey due to the H264 encoding
* `preview-opacity=50` means you can still use your RPi display
  * especially useful when using CLI mode
* See [gstreamer rpicamsrc](https://gstreamer.freedesktop.org/documentation/rpicamsrc/index.html?gi-language=c) docs for more properties

An alternative to `rpicamsrc` is `v412src`; but, often frames get dropped doing this because of the H264 encoding.

```sh
gst-launch-1.0 v4l2src device=/dev/video0 ! \
video/x-h264,width=640,height=480,framerate=30/1 ! \
h264parse ! \
rtph264pay config-interval=1 pt=96 ! \
gdppay ! \
tcpserversink host=0.0.0.0 port=5000
```

## Watch the stream using gstreamer

```sh
gst-launch-1.0 -v tcpclientsrc host=192.168.50.202 port=5000 ! \
gdpdepay ! rtph264depay ! avdec_h264 ! videoconvert ! \
autovideosink sync=false
```

`sync=false` means we always get the latest frame. This helps keep the latency as low as possible.

