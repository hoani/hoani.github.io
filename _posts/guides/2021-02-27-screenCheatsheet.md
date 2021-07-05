---
title: "Screen cheatsheet"
excerpt: "Simple screen command reference"
toc: true
categories:
  - guide
  - software-guide
---

## Command Line

```sh
screen -S name  # start a session called name
screen -r name  # reattach to name
```

## Screen Commands

Terminal region splitting and navigation

```sh
clt-a |    # split verical regions
ctl-a S    # split horizontal regions
clt-a X    # close the current region
clt-a tab  # move to next region
```

Window management

```sh
ctl-a c    # create a window
clt-a "    # navigate to a window 
ctl-a :    # save current layout
  `layout save default`
ctl-a :    # resize horizonal region to 100 characters
  `resize -h 100`
ctl-a :    # resize vertical region to 20 lines
  `resize -v 20`
```

General management

```sh
ctl-a esc  # access scroll buffer
  esc      # exit scroll buffer
ctl-a d    # detach
ctl-a k    # kill session
```