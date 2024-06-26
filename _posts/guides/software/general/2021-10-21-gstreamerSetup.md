---
title: "GStreamer Setup"
excerpt: "Install and Run GStreamer on Debian Distros"
toc: true
permalink: /guides/software/gstreamer
categories:
  - guide
  - software
  - gstreamer
---

This guide was tested on Linux Mint and a Raspberry Pi.

## GStreamer

Is a library with a set of tools for multimedia. 

## Installation

Following a [LCA2018 Gstreamer tutorial](https://www.youtube.com/watch?v=ZphadMGufY8):

```
sudo apt-get install gstreamer1.0-dev gstreamer1.0-tools \
gstreamer1.0-plugins-\* gstreamer1.0-libav
```

## Hello world

Audio:

```
gst-launch-1.0 audiotestsrc ! audioconvert ! autoaudiosink
```

Video: 

```
gst-launch-1.0 videotestsrc ! videoconvert ! autovideosink
```

I have found that gstreamer drops a lot of frames on VMs. For testing on VMs, use `videotestsrc`.

## With a camera

With a webcam or USB camera:

```
gst-launch-1.0 v4l2src device=/dev/video0 \
! videoconvert ! autovideosink
```

When using a raspberry pi camera, we will want to specify framerate and resolution:

```
gst-launch-1.0 v4l2src device=/dev/video0 \
! video/x-raw,width=640,height=480,framerate=30/1 \
! videoconvert ! autovideosink
```

