---
title: "Context in Go"
excerpt: "Managing Go routines using the Context package"
permalink: /guides/software/golang/context
toc: true
categories:
  - guide
  - golang
  - golang-guide
---

The [context package](https://golang.org/pkg/context) standardizes a way of running go routines which are cancellable. Each `context.Context` object can have children, and when a parent is cancelled, so are its children. 

## Syntax

A go routine with context has a context object as its first argument:

```go
go heartbeatMessage(ctx, d, msg)
```

## Creating Context Objects

Context objects represent a tree, the root of the tree is the Background:
```go
ctx := context.Background()
```

From the background context, we can create:

```go
ctx, cancel := context.WithCancel(ctx)
ctx, cancel := context.WithTimeout(ctx, time.Second)
```

If a timeout is specified, the context will automatically be cancelled when the timeout expires.

*Note: `cancel` must always be called regardless of whether the timeout is triggered. We can safely call `cancel` multiple times; only the first call matters. A good pattern is to use defer:*

```go
ctx, cancel := context.WithCancel(ctx)
defer cancel()
```


## Determining if a context has been cancelled

The go routine determines when it is cancelled by checking:

```go
<-ctx.Done()
```

## Simple example

The following is a simple heartbeat which either times out or gets cancelled by context.

```go 
package main

import (
	"context"
	"fmt"
	"time"
)

func heartbeatMsg(ctx context.Context, 
                  d time.Duration, msg string) {
	for {
		select {
		case <-time.After(d):
			fmt.Println(msg)
		case <-ctx.Done():
			fmt.Println("Ending heartbeat")
			return
		}
	}
}

func main() {
	ctx := context.Background()
	ctx, cancel := context.WithTimeout(ctx, 2*time.Second)
	defer cancel()
	go heartbeatMsg(ctx, 200*time.Millisecond, "ba-dum")
	select {
	case <-time.After(1*time.Second):
		fmt.Println("Cancel context")
		cancel()
	case <-ctx.Done():
		fmt.Println("Context timeout triggered")
	}
	// let heartbeat end properly
	<-time.After(time.Millisecond) 
}
```


