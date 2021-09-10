---
title: "Raspberry Pi C++ IO"
excerpt: "Controlling Raspberry Pi IO with C++"
toc: true
categories:
  - guide
  - software
  - software-guide
  - electronics
  - electronics-guide
  - cpp-guide
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

## pigpio

The [pigpio](http://abyz.me.uk/rpi/pigpio/cif.html) library provides a C API to many common Raspberry Pi interfaces.

### Install

On Raspberry Pi OS:

```sh
sudo apt update
sudo apt install pigpio
```

### Some Examples

See [PiGPIO C Interface](http://abyz.me.uk/rpi/pigpio/cif.html) for full interface.

The library also includes serial, SPI and i2c libraries (not covered here).

#### Turning on an LED

```cpp
#include <pigpio.h>

int main(int argc, char *argv[]) {
  if (gpioInitialise() < 0) {
    return 1; // Failed to initialize.
  }

  gpioSetMode(26, PI_OUTPUT);

  gpioWrite(26, PI_HIGH);
  time_sleep(1.0);
  gpioWrite(26, PI_LOW);

  gpioTerminate(); // Compulsary.
  return 0;
}
```

#### Reading from a Button

```cpp
#include <stdio.h>
#include <pigpio.h>

int main(int argc, char *argv[]) {
  if (gpioInitialise() < 0) {
    return 1;
  }

  gpioSetMode(21, PI_INPUT);
  gpioSetPullUpDown(21, PI_PUD_UP);

  if (gpioRead(21) == PI_LOW) {
    fprintf(stdout, "<PRESSED>\n");
  } else {
    fprintf(stdout, "<NONE>\n");
  }

  gpioTerminate(); // Compulsary.
  return 0;
}
```

#### Commanding a Servo

```cpp
#include <pigpio.h>

int main(int argc, char *argv[]) {
  if (gpioInitialise() < 0) {
    return 1; // Failed to initialize.
  }

  const double start = time_time();
  while ((time_time() - start) < 60.0) {
    time_sleep(0.5);
    gpioServo(24, 1500);

    time_sleep(0.5);
    gpioServo(24, 1800);
  }

  gpioTerminate(); // Compulsary.
  return 0;
}
```

* If `gpioServo(24, 0)` is called, then it receives no power

#### Pin Interrupts

```cpp
#include <stdio.h>
#include <pigpio.h>

void triggered(int gpio, int level, uint32_t tick) {
  if (level == 0) {
    fprintf(stdout, "Pressed");
  } else if (level == 1) {
    fprintf(stdout, "Released");
  } else {
    fprintf(stdout, "Timeout");
  }
  fprintf(stdout, "at %f s\n", (double)tick/1e6);
}

int main(int argc, char *argv[]) {
  if (gpioInitialise() < 0) {
    return 1;
  }

  gpioSetMode(21, PI_INPUT);
  int timeout_ms = 1000; // Set to 0 to disable.
  gpioSetISRFunc(21, EITHER_EDGE, timeout_ms, triggered);

  time_sleep(60.0);

  gpioTerminate(); // Compulsary.
  return 0;
}
```

* The timeout allows us to trigger events when no edge change is detected.
* The `level` values are:
  * 0 = Falling Edge
  * 1 = Rising Edge
  * 2 = Watchdog Timeout (optional)

#### Potentially Useful .bashrc Shortcuts

```sh
# Compiles and runs a pigpio c file.
hpigpio_c(){
  mkdir -p bin
  gcc -o ./bin/${1//.c} $1 -lpigpio -lrt -lpthread
  sudo ./bin/${1//.c}
}

# Compiles and runs a pigpio c++ file.
hpigpio_cpp(){
  mkdir -p bin
  g++ -Wall -pthread -o bin/${1//.cpp} $1 -lpigpio -lrt 
  sudo ./bin/${1//.cpp}
}
```
