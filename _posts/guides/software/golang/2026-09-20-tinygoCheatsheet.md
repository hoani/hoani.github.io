---
title: "Tinygo cheatsheet"
excerpt: "Cheatsheet for tinygo"
toc: true
permalink: /guides/software/golang/tinygo
categories:
  - guide
  - golang
  - tinygo
  - software
---

Tinygo compiles go for various microcontrollers.

Useful links:
* [Supported Microcontrollers](https://tinygo.org/docs/reference/microcontrollers/)
* [Machine Package](https://tinygo.org/docs/reference/machine/)

## Command Line

When using the command line, we provide `--target` to determine which board/microprocessor we are using

```sh
tinygo build --target=pico ./cmd/hello
tinygo flash --target=pico ./cmd/hello
```

If your board has a serial monitor present:

```sh
tinygo monitor
```

## VsCode IDE setup

For better intellisense and compiler warnings, install the `TinyGo` extension. Then select your target from the bottom ribbon. 

This will generate a `.vscode/settings.json` file for you with the correct setup for your board.

## Conditional Compilation

For a primer see [Golang Conditional Compilation](/guides/software/golang/conditional-compilation)

When your have a source `.go` file which should only be compiled for tinygo, use:
```
//go:build tinygo
```

A `tinygo` project which targets multiple processors will want to compile different source files depending on which processor is being targetted.

For example, when targetting a `pico` board use:

```
//go:build tinygo && pico
```

The specific flag to use can be found in the [tinygo/src/machine package](https://github.com/tinygo-org/tinygo/tree/dev/src/machine). Find the `board_` or `machine_` go file which matches your target and see which flag is being used.

## Hello World

Note that each microcontroller defines GPIOs differently, the same can be true for other hardware peripherals too.

When supporting multiple boards, it can be useful to define a hardware abstraction layer to abstract these details behind a common interface which is conditionally compiled in.

```go
package main

import (
  "machine"
  "time"
)

func main() {
  pin := machine.D13   // Teensy
  pin := machine.GP13  // Pico
  led := machine.P0_13 // NRF52840
  led.Configure(machine.PinConfig{Mode: machine.PinOutput})
  for {
    led.Low()
    time.Sleep(time.Millisecond * 500)
    led.High()
    time.Sleep(time.Millisecond * 500)
  }
}
```
