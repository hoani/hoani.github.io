---
title: "Protobuf Golang"
excerpt: "Simple encoding/decoding in Golang"
toc: true
permalink: "/guides/software/golang/protobuf"
categories:
  - guide
  - golang
  - golang-guide
---

## Setup

See [Protobuf Basics](/guides/software/protobuf)

Create a new go module to work in:

```sh
go mod init github.com/hoani/robot
go get google.golang.org/protobuf/proto
```

## Protobuf message

```java
// robot.proto
syntax = "proto3";
option go_package = "github.com/hoani/robot";

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
protoc --go_out=proto --go_opt=paths=source_relative robot.proto
```

## Encoding and Decoding

```go
// main.go
package main

import (
	"fmt"

	robot "github.com/hoani/robot/proto"
	"google.golang.org/protobuf/proto"
)

func main() {
	g := &robot.GimbalState{
		ServoPitch: &robot.ServoState{Angle: 10.0},
		ServoYaw:   &robot.ServoState{Angle: -15.5},
	}
	fmt.Printf("Gimbal  \tPitch: %v \tYaw: %v\n",
		g.ServoPitch.Angle,
		g.ServoYaw.Angle)

	encoded, _ := proto.Marshal(g)
	fmt.Printf("Encoded \tBytes: %v\n", len(encoded))

	decoded := &robot.GimbalState{}
	proto.Unmarshal(encoded, decoded)

	fmt.Printf("Decoded \tPitch: %v \tYaw: %v\n",
		decoded.ServoPitch.Angle,
		decoded.ServoYaw.Angle)
}

```

Running the example:
```
└─ $ ▶ go run ./main.go
Gimbal          Pitch: 10       Yaw: -15.5
Encoded         Bytes: 22
Decoded         Pitch: 10       Yaw: -15.5
```

