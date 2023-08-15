---
title: "Configuring Cameras with v4l2-ctl"
excerpt: "Camera config for Raspberry Pi/Ubuntu distros"
permalink: /guides/software/raspberrypi/gstreamer
toc: true
categories:
  - guide
  - software
  - rpi
---

You can get an overview of cameras available using:
```
v4l2-ctl --all
```

If you want to know which `/dev/video<n>` maps to your device:
```
v4l2-ctl --list-devices
```

From here on, we assume the camera in question is on `/dev/video0`.

To learn what formats, resolution and frame-rate options your camera provides:
```
v4l2-ctl --device=/dev/video0 --list-formats-ext
```

From here, the camera's settings can be updated, for example, the following sets resolution:
```
 v4l2-ctl --device=/dev/video0 --set-fmt-video=width=1280,height=960 --verbose
```

Or you can change the encoding to one of the numeric indexes shown in `--list-formats-ext`:
```
v4l2-ctl --device=/dev/video0 --set-fmt-video=pixelformat=2 --verbose
```

Additional available controls are shown with:
```
v4l2-ctl --device=/dev/video0 --list-ctrls
```








