---
title: "Protobuf Basics"
excerpt: "Basics for creating and using protobuffers"
toc: true
permalink: /guides/software/protobuf
categories:
  - guide
  - software
  - software-guide
---

## Installation

If you want to use protobuffers with golang:
```
go install google.golang.org/protobuf/cmd/protoc-gen-go@latest
```

Otherwise:

Ubuntu
```
sudo apt-get install protobuf-compiler
```

Mac OS
```
brew install protobuf
```

## Make a .proto file

```java
// robot.proto
syntax = "proto3";

package robot;

message ServoState {
  double angle = 1;
}

message GimbalState {
  ServoState servoPitch = 1;
  ServoState servoYaw = 2;
}
```

## Generate Protobuffer files

```
protoc --python_out=$OUT_DIR --$SRC_DIR/robot.proto
```

Alternative output options are:
* `--go_out` (with golang installation only)
* `--cpp_out`, `--csharp_out`, `--java_out`
* `--js_out`, `--objc_out`, `--php_out`, `--ruby_out`

## Cheatsheet for .proto files

### Types

Floating point
```
float, double
```

Integers:
```
int32, int64
uint32, uint64
sint32, sint64
fixed32, fixed64
sfixed32, sfixed64
```
* use `sint` over `int` for positive and negative integer representation
* small integers using `32` integers will usually be represented by less than 4 bytes
  * the exception to this are the `fixed` representations

Boolean
```
bool
```

Strings
```
string
```

Byte vector (up to 2^32 bytes)
```
bytes
```

### Special types and structures

Zero or more entries (sort of like a vector):
```
repeated uint32 ledPins = 1;
```

Enumerations
```
enum Task {
  UDPATE_ATTITUDE = 0;
  UPDATE_MOVEMENT = 1;
  HANDLE_INPUTS = 2;
}
```

Maps:
```
map<Task, uint32> updateRatesUs = 1;
```

### Unique identifiers

The `= 1;` value is a unique identifier for that field. Each field must have a distinct identifier.

This enables future compatability between protobuffers as a protocol may add or remove fileds.

See [Update a message type](https://developers.google.com/protocol-buffers/docs/proto3#updating) for more details on backwards compatability.

### Importing other .proto files

```java
import "google/protobuf/timestamp.proto";

...

message GimbalState {
  ServoState servoPitch = 1;
  ServoState servoYaw = 2;
  google.protobuf.Timestamp lastUpdate = 3;
}
```

See [google.protobuf](https://developers.google.com/protocol-buffers/docs/reference/google.protobuf) has a bunch of useful types - most of which can be found in the [github source](https://github.com/protocolbuffers/protobuf/tree/master/src/google/protobuf). 

See `wrappers.proto` for primitive types.
 

