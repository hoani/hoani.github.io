---
title: "Raspberry Pi Python IO"
excerpt: "Controlling Raspberry Pi IO with Python"
toc: true
categories:
  - guide
  - software
  - software-guide
  - electronics
  - electronics-guide
  - python-guide
---

## Enable Hardware

On Raspberry Pi OS-Desktop:
* access the start menu 
* `Preferences->Raspberry Pi Configuration`

On the command line: 
* `sudo raspi-config`
* enable/disable various IO in interface options

## Check Pinout

Use `pinout` to identify GPIO pin numbers from terminal.

## GPIO Zero

The [GPIO Zero](https://gpiozero.readthedocs.io/en/stable/index.html) library provides an API to many common Raspberry Pi interfaces.

### Install

On Raspberry Pi OS:

```sh
sudo apt update
sudo apt install python3-gpiozero
```

### Some Examples

See [GPIO Zero recipes](https://gpiozero.readthedocs.io/en/stable/recipes.html) for many more examples.

#### Turning on an LED

```py
from gpiozero import LED
from time import sleep

led = LED(26)
led.on()
sleep(1)
```

#### Reading from a Button

`Button` is by default a switch which pulls the input low when active. See [button docs](https://gpiozero.readthedocs.io/en/stable/api_input.html#button) for more options.

```py
from gpiozero import Button

button = Button(21)

if button.is_pressed:
    print("<Pressed>")
else:
    print("<None>")
```

#### Commanding a Servo

* [`Servo`](https://gpiozero.readthedocs.io/en/stable/api_output.html#servo) has range -1.0 to 1.0
* [`AngularServo`](https://gpiozero.readthedocs.io/en/stable/api_output.html#servo) has range from -90 to 90 (but can be changed)

```py
from gpiozero import AngularServo
from time import sleep

servo = Servo(18)

while True:
    servo.angle = 45
    sleep(2)
    servo.angle = -45
    sleep(2)
```

* If the `servo.angle = None`, then it receives no power

#### Reading from an Encoder

```py
from gpiozero import RotaryEncoder
from time import sleep

# Note: For https://www.dfrobot.com/product-1431.html
encoder = RotaryEncoder(5, 6, max_steps=105, wrap=True)

while True:
    print("encoder position {}".format(encoder.value*180))
    sleep(1)
```

#### Generic Output Devices

```py
from gpiozero import DigitalOutputDevice
from time import sleep

dout = DigitalOutputDevice(26)

while True:
    sleep(1)
    dout.off()
    sleep(1)
    dout.on()
```

#### Generic Input Devices

```py
from gpiozero import DigitalInputDevice
from time import sleep

din = DigitalInputDevice(21, pull_up=True)
din.when_activated=lambda: print("HIGH")
din.when_deactivated=lambda: print("LOW")

while True:
    sleep(1)
    if din.value:
        print("{} seconds".format(din.active_time))
    else:
        print("{} seconds".format(din.inactive_time))
```
