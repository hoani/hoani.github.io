---
title: "Protobuf Python"
excerpt: "Simple encoding/decoding in Python"
toc: true
permalink: /guides/software/python/protobuf
categories:
  - guide
  - python
  - python-guide
---

## Setup

See [Protobuf Basics](/guides/software/protobuf)

## Protobuf message

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

## Compile

```
protoc --python_out=. ./robot.proto
```

## Encoding and Decoding

```py
import robot_pb2
gimbal = robot_pb2.GimbalState()
gimbal.servoPitch.angle = 10.0
gimbal.servoYaw.angle = -15.5

encoded = gimbal.SerializeToString()
print(len(encoded))   # 22

decoded = robot_pb2.GimbalState()
decoded.ParseFromString(encoded)
print(decoded.servoYaw.angle)   # 10.0 -15.5
```

Note:
* `decoded.ParseFromString` returns `22` which can be used to parse streams.
