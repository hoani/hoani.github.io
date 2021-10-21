---
title: "GStreamer Setup"
excerpt: "Install and Run GStreamer on Debian Distros"
toc: true
categories:
  - guide
  - software
  - software-guide
---

This guide was tested on Linux Mint and a Raspberry Pi.

## GStreamer

Is a library with a set of tools for multimedia. 

## Installation

Following a [LCA2018 Gstreamer tutorial](https://www.youtube.com/watch?v=ZphadMGufY8):

```
sudo apt-get install gstreamer1.0-dev gstreamer1.0-tools gstreamer1.0-plugins-\* gstreamer1.0-libav
```

## Hello world

With a webcam or USB camera:

```
gst-launch-1.0 v4l2src device=/dev/video0 ! videoconvert ! xvimagesink
```

If you don't have a camera:

```
gst-launch-1.0 videotestsrc ! videoconvert ! xvimagesink
```

I have found that gstreamer drops a lot of frames on VMs. For testing on VMs, use `videotestsrc`.

