---
title: "Python Serial Tools"
excerpt: "Useful terminal-based serial tool"
toc: true
permalink: /guides/software/python/miniterm
categories:
  - guide
  - python
  - python-guide
  - electronics
---

I recently came across a nice way of dumping serial data into a log file using `pyserial`.

If it isn't installed yet:

```sh
pip3 install pyserial
```

## Dumping data to a log file

With the teensy attached to my PC, I can connect using:

```sh
python3 -m serial.tools.miniterm /dev/tty.usbmodem39153401 \
115200 > log.csv
```

The output looks like:

```sh
--- Miniterm on /dev/tty.usbmodem39153401  115200,8,N,1 ---
--- Quit: Ctrl+] | Menu: Ctrl+T | Help: Ctrl+T followed by Ctrl+H ---

--- exit ---
```

So `CTRL + ]` will stop the logging.

The generalized command is:

```sh
python3 -m serial.tools.miniterm <PORT> <BAUD> > <LOGFILE>
```

We can also not direct data to the log file if we want to just use the miniterm.

### List Ports

Another helpful utility to check what ports we can actually direct serial to is `list_ports`

```sh
python3 -m serial.tools.list_ports -v
```

Which outputs:

```sh
/dev/cu.Bluetooth-Incoming-Port
    desc: n/a
    hwid: n/a
/dev/cu.usbmodem39153401
    desc: USB Serial
    hwid: USB VID:PID=16C0:0483 SER=3915340 LOCATION=20-3
2 ports found
```

The `-v` flag can be omitted to just show the port names.




