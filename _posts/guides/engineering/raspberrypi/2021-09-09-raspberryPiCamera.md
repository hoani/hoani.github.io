---
title: "Raspberry Pi Camera"
excerpt: "Setting Up the PiCam on Raspberry Pi"
toc: true
permalink: /guides/software/raspberrypi/picam
categories:
  - guide
  - software
  - software-guide
  - electronics
  - electronics-guide
  - python-guide
  - rpi-guide
---

## Enable Camera

On Raspberry Pi OS-Desktop:
* access the start menu 
* `Preferences->Raspberry Pi Configuration`

On the command line: 
* `sudo raspi-config`
* enable/disable the camera in interface options

## Verify Camera

`vcgencmd` allows us to query the [VideoCore GPU](https://www.raspberrypi.org/documentation/computers/os.html#vcgencmd) - we can use this to verify our camera has been detected:

```sh
vcgencmd get_camera # supported=1 detected=1 when successful
```

If this is unsuccessful, the PiCam cable may not be connected properly.

## Take a Photo

```sh
raspistill -o image.jpg
```

<figure>
    <img src="/assets/images/posts/guides/rpi/000_picam.jpg">
</figure>

Run `raspistill --help` to see all of the additional options and filtering available.

Another tool [`raspiyuv`](https://www.raspberrypi.org/documentation/accessories/camera.html#raspiyuv) generates uncompressed images which can be useful for computer vision.

## Take a Video

```sh
raspivid -o vid.h264 -t 10000
```

The `-t` flag sets how long to record in milliseconds
* default is `5000` milliseconds
* set to `0` to record until cancelled

## Streaming

Assuming we have a raspberry pi on `192.168.1.123`, and we would like to stream on port `5000`:

```sh
raspivid -t 0 -w 320 -h 240 --inline --listen -o tcp://0.0.0.0:5000
```

On client side:
```
vlc tcp/h264://192.168.1.123:5000
```

I have found that gstreamer has significantly lower latency than using `raspivid` directly. See [GStreamer RPi Streaming](https://hoani.net/posts/guides/2021-10-22-gstreamerRPiStreaming/).

## Python pycamera Library

Install the [pycamera library](https://picamera.readthedocs.io/) on Raspberry Pi OS:

```sh
sudo apt update
sudo apt install python3-pycamera
```

### Capturing an Image

```py
from picamera import PiCamera

camera = PiCamera()
camera.capture('image.jpg')
```

### Recording a Video

```py
from picamera import PiCamera

camera = PiCamera()
camera.start_recording('video.h264')
camera.wait_recording(5)
camera.stop_recording()
```

See the [Basic Recipies Documentation](https://picamera.readthedocs.io/en/release-1.13/recipes1.html) for more.







