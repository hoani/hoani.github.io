---
title: "Raspberry Pi Python IO"
excerpt: "Controlling Python IO with Raspberry Pi"
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

If you're running the Raspberry Pi with Raspbian-Desktop, you can reach this config from `start->Preferences->Raspberry Pi Configuration`.

On the command line, use `sudo raspi-config`. Then in interface options, you can enable/disable hardware interfaces that you want to support.

## Check Pinout

Actual pinouts change between different revisions of the Raspberry Pi.

From the terminal: `pinout`

## PiCamera

The PiCamera library lets

## GPIO Zero

The [GPIO Zero](https://gpiozero.readthedocs.io/en/stable/index.html) library provides an API to many common Raspberry Pi interfaces.

### Install

On Raspbian:

```sh
sudo apt update
sudo apt install python3-gpiozero
sudo apt install python3-pycamera
```
