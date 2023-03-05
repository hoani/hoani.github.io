---
title: "Go cheatsheet"
excerpt: "Cheatsheet for golang"
toc: true
permalink: /guides/software/golang/cheatsheet
categories:
  - guide
  - golang
  - software
---

## Command Line

```sh
go build
go run
go test
```

## Loop Syntax

```go
for { // forever loop
}
```
```go
for condition { 
}
```
```go
for i, value := range slice { // works for maps too
}
```
```go
for i:=0; i<N; i++ {
}
```

## Slices Syntax

```go
fibonacci := [10]int{0, 1, 1, 2, 3, 5, 8, 13, 21, 34}
var slice []int = fibonacci[3:7] // takes indexes 3, 4, 5, 6
slice2 := make([]int, 5)
fmt.Println(slice) // [2 3 5 8]
```
```go
slice[0] = 1337        // note: slices are references!
fmt.Println(fibonacci) // [0, 1, 1, 1337, 3, 5, 8, 13, 21, 34]
```
```go
slice = append(slice, element, ...)
```

## Map syntax

```go
myMap := make(map[string]float32)
```
```go
myMap["Accel"] = 9.81
```
```go
// (0.0, false) if no "gyro" field; (val, true) otherwise
rate, has_field := myMap["Gyro"] 
```

## Object Oriented-like things

### Adding methods to structs

- `(v Vector3)` is known as a receiver, it copies the data
- `(v *Vector3)` is a pointer receiver, and is useful when we will modify the struct

```go
type Vector3 struct {
  X, Y, Z float64
}
func (v *Vector3) Norm() float64 {
  return math.Sqrt(v.X*v.X + v.Y*v.Y + v.Z*v.Z)
}
func (v *Vector3) Add(a float64) {
  v.X += a
  v.Y += a
  v.Z += a
}
distance = Vector3{1.0, 2.0, 3.0}.Norm()
```

### Interfaces

Interface implementation is implicit, if a struct has the interface's method signatures, it implements the interface.

```go
type Vector interface {
  Norm() float64
  Add() float64
}

var v Vector
v3 := Vector3{0.0, 0.0, 0.0}
v := &v3
```

Type assertions allow us to check the underlying type of something whcih meets an interface:
```go
val, ok := v.(Vector3) // ok = true
val, ok := v.(int32)   // ok = false
```

Type switches:
```go
switch v := i.(type) {
case Vector3:
  // ...
case Vector2:
  // ...
default:
  // ...
}
```

## Concurrency 

Run a go routine
```go
func doSomething() {
}
go doSomething()
```

### Channels

```go
ch := make(chan float64)
ch := make(chan float64, 100) // buffer 100 entries
```

```go
ch <- value // send to channel
value := <- ch  // receive from channel
```

```go
func doSomething(ch chan float64)
```

```go
close(ch) // closes the channel
```

```go
value, ok := <- ch // ok = false if ch is closed
```

```go
// loop until the channel closes 
for value  := range ch {
}
```

```go
select { // blocks until one of its case can be run
case ch1 <- x: // runs when we can load data into c
case <-ch2:    // runs when ch2 has data
}
```
```go
select { // non-blocking
case ch1 <- x: // runs when we can load data into c
case <-ch2:    // runs when ch2 has data
default:       // runs if nothing is ready
}
```
```go
time.Tick(Duration)  // periodic channel
time.After(Duration) // periodic channel
```

## Enums

Use `iota`:

```go
type Commands int

const (
    Sit Commands = iota
    Fetch
    RollOver
)
```

This [stack exchange answer](https://stackoverflow.com/questions/14426366/what-is-an-idiomatic-way-of-representing-enums-in-go) details some more advanced techniques including bitfield enumeration.