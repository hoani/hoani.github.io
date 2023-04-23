---
title: "Go Functional Options Pattern"
excerpt: "Instantiating structs with the options pattern."
toc: true
permalink: "/guides/software/golang/options"
categories:
  - guide
  - golang
  - software
---

The functional options pattern is a popular way to instantiate structs in Go.

It has two parts:
* `New(options ...func(*MyStruct)) *MyStruct`
* Options `WithSomething(s Something) func(*MyStruct)`

Option functions return a function which configures your structure, they typically look like:

```go
func WithSomething(s Something) func(*MyStruct) {
  return func(m *MyStruct) {
    m.something = s
  }
}
```

Then the inside of your `New` function looks like:
```go
func New(options ...func(*MyStruct)) *MyStruct {
  // Default config here.
  m := &MyStruct{}
  // ...

  // Apply options.
  for _, option := range options {
    option(m)
  }

  return m
}
```

## Example: Http Client

```go
package main

import (
  "fmt"
  "net/http"
  "time"
)

type Option func(*http.Client)

func New(opts ...Option) *http.Client {
  c := &http.Client{
    Timeout: 5 * time.Second,
  }
  for _, opt := range opts {
    opt(c)
  }
  return c
}

func WithTimeout(t time.Duration) func(*http.Client) {
  return func(c *http.Client) {
    c.Timeout = t
  }
}

func main() {
  c := New(WithTimeout(time.Second))
  res, err := c.Head("http://hoani.net")
  if err != nil {
    panic(err)
  }
  fmt.Printf("status %v\n", res.Status)
}
```
