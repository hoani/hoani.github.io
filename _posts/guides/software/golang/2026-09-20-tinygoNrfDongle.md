---
title: "Tinygo - Bluetooth with nrf52840 dongle"
excerpt: "Flashing the nrf52840 dongle with a bluetooth example"
toc: true
permalink: /guides/software/golang/tinygo-nrfdongle
categories:
  - guide
  - golang
  - tinygo
  - bluetooth
  - software
---

The nrf52840 dongle is a small cheap device which I feel is perfect for hobby projects.

`tinygo` has strong bluetooth support for nrf devices.

However, getting the nrf52840 dongle flashed takes a few steps, this document is what worked for me.

## Prerequisites

* `tinygo` and `go`
* Install [nrfutil](https://docs.nordicsemi.com/r/bundle/nrfutil/page/get-started)
* dongle must still have its factory bootloader intact

Clone the bluetooth repository, this gives us access to examples, but also the required SoftDevice binary:
```sh
git clone https://github.com/tinygo-org/bluetooth.git
cd bluetooth
```

## First Flash with SoftDevice

The first time we flash our dongle, we need to combine our binary with the SoftDevice binary to access bluetooth functionality

### Build Our App

Build against `pca10059-s140v7`. 

```sh
tinygo build -target=pca10059-s140v7 -o app.hex ./examples/heartrate
```

* `pca10059-s140v7` links the application above the S140 v7 SoftDevice region.
  * note: if we targeted `pca10059`, the application would be placed where the SoftDevice region would be, so `pca10059` is incompatable with the `bluetooth` package.
* We use `.hex` instead of `.bin` to specify to the DFU tool where to place the binary

### Package with SoftDevice

The `tinygo-org/bluetooth` repository includes the SoftDevice hex. Find it's location in the repository, I found it in `s140_nrf52_7.3.0/s140_nrf52_7.3.0_softdevice.hex`.

Package the binaries together using `nrfutil`:

```sh
nrfutil nrf5sdk-tools pkg generate \
  --hw-version 52 \
  --sd-req 0x00 \
  --sd-id 0x0123 \
  --softdevice s140_nrf52_7.3.0/s140_nrf52_7.3.0_softdevice.hex \
  --application app.hex \
  --application-version 1 \
  fw.zip
```
* `--hw-version 52` - `52` is Nordic's convention for nrf52 boards
* `--sd-req 0x00` - indicates no SoftDevice is currently on target
* `--sd-id 0x0123` - the firmware ID of the SoftDevice being used
  * This references the s140 v7.3.0 firmware ID
  * You can see the list of all firmware IDs by calling `nrfutil nrf5sdk-tools pkg generate --help`

### Flash the Dongle

Find which port the dongle has connected to. It can be helpful to use `nrfutil device list` - if the device package isn't installed, `nrfutil install device` first:

```sh
nrfutil device list   
  0
  Product         USB2.0 Hub             
  Ports           /dev/tty.usbmodem13101
  Traits          nordicUsb, serialPorts, usb

  Found 1 supported device
```

Once we know the dongle's port, we can flash it:

```
nrfutil nrf5sdk-tools dfu usb-serial --package fw.zip --port /dev/tty.usbmodem13101
```

## Flashing Without SoftDevice

Packaging and flashing with SoftDevice only needs to be done once.

For subsequent updates:

```sh
tinygo build -target=pca10059-s140v7 -o app.hex ./examples/heartrate

nrfutil nrf5sdk-tools pkg generate \
  --hw-version 52 \
  --sd-req 0x0123 \
  --application app.hex \
  --application-version 1 \
  app.zip

nrfutil nrf5sdk-tools dfu usb-serial --package app.zip --port /dev/ttyACM0
```

Note, we now use `--sd-req 0x0123` - we require that this soft device is loaded.
