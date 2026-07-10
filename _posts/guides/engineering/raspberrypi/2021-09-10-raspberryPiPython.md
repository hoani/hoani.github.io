---
title: "Raspberry Pi Python IO"
excerpt: "Controlling Raspberry Pi IO with Python"
permalink: /guides/software/raspberrypi/python-io
toc: true
categories:
  - guide
  - software
  - electronics
  - python
  - rpi
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

## Reducing Servo Jitter

When running the servo code above, we get the warning:

```sh
PWMSoftwareFallback: To reduce servo jitter, use the pigpio pin factory.
```

We would like our PWM to be hardware driven, rather than be at the mercy of the OS.

There are two steps:
* Install `pigpio`
* Use GPIO Zero pin factories

### Installing pigpio

On Raspberry Pi OS:

```sh
sudo apt update
sudo apt install pigpio
sudo apt install python3-pigpio
```

#### pigpio daemon

`pigpio` uses a daemon (that takes port `:8888`) to control IO pins. 

The daemon must be running to control IO pins from your Raspberry Pi.

Start the pigpio daemon:
```sh
sudo systemctl start pigpiod
```

Enable pigpio daemon at boot:
```sh
sudo systemctl enable pigpiod
```

### Using a Pin Factory

GPIO Zero uses [Pin Factroies](https://gpiozero.readthedocs.io/en/stable/api_pins.html) to provide various pin drivers.

Once `pigpio` is installed, we can use its pin factory.

```py
from gpiozero import Servo
from gpiozero.pins.pigpio import PiGPIOFactory
from time import sleep

factory = PiGPIOFactory()
servo = Servo(18, pin_factory=factory)

while True:
    servo.min()
    sleep(2)
    servo.mid()
    sleep(2)
    servo.max()
    sleep(2)
```

This time, the pulses are very stable and the servo has no jitter.

### Jitter Comparison

I wanted to compare `pigpio` to the default pin driver; so I put a scope on the two implementations:

<figure>
    <img src="/assets/images/posts/guides/rpi/100_pwmFactory.png">
</figure>

* yellow is `pigpio` 
  * on closer inspection only jitters about 0.2 microseconds; error (0.02%).
* blue is the default driver
  * it appears to put out two pulses per cycle
  * the second pulse seems to be asyncronous and the average of the two provide the servo pulse

## pigpio

GPIO Zero doesn't support some IO (such as serial).

In these cases, we can interface with the pigpio daemon directly.

See the [pigpio python API](https://abyz.me.uk/rpi/pigpio/python.html) for more details.

### Interfacing with a serial port

Assuming `pigpio` is already installed and the daemon is running:

```py
import pigpio
import time

pi = pigpio.pi()
if not pi.connected:
  exit()

s = pi.serial_open("/dev/ttyS0", 115200)
pi.serial_write(s, "j")

time.sleep(1)
len, data = pi.serial_read(s, 100)

if len > 0:
  print(data.decode("utf-8"))
else:
  print("no data")
```

* This interfaces with a [bittle](https://bittle.petoi.com/4-configuration#4-4-raspberry-pi-serial-port-as-an-interface) on a Raspberry Pi 3
  * Other models of Raspberry Pi may use `/dev/ttyAMA0`
* When debugging serial ports, consider [PySerial's miniterm](https://hoani.net/posts/guides/2020-12-10-pythonMiniterm/)
