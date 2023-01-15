---
title: "Go Conditional Compilation"
excerpt: "Using different source code for different machines."
toc: true
permalink: "/guides/software/golang/conditional-compilation"
categories:
  - guide
  - golang
  - golang-guide
---

{% include video id="mIfzwTfXnWI" provider="youtube" %}

We sometimes find ourselves in situations where we need to compile different source files depending on the target operating system or chipset.

There are two common ways to selectively compile go code:
* File suffixes
* Build tags

## File suffixes

Say we have a function called `PrintMyOs` that we want to compile to output the name of our operating system.

We can define `PrintMyOs` in various `printer_<os>.go` files:
* `printer_linux.go` - Linux
* `printer_darwin.go` - MacOS
* `printer_windows.go` - Windows
* `printer_android.go` - Android

The general trick is that any valid `GOOS` option (see [syslist](https://github.com/golang/go/blob/master/src/go/build/syslist.go)) can be used as a file extension.

The go compiler picks up the suffix of the filename and will only compile that file if the target `GOOS` is that operating system.

## Build tags

Sometimes if we are writing some kind of go driver, using file suffixes isn't enough. 

If we want to consider the chipset architectures as well, go provides another conditional compilation technique called build tags:
* To only build a file for Linux:
  ```go
  // +build linux
  ```
* To build for any operating system which isn't Linux:
  ```go
  // +build !linux
  ```
* To build only on MacOs for arm architecture (e.g. M1 macbooks):
  ```go
  // +build darwin
  // +build arm
  ```

A more modern (1.17 and higher) build tag uses a `//go:build` syntax, with boolean expressions.

To make code compile on any machine which isn't an M1 macbook:
```go
//go:build !(darwin && arm)
```
