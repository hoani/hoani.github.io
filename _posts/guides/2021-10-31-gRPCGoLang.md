---
title: "Go gRPC"
excerpt: "Basic gRPC using Go"
toc: true
categories:
  - guide
  - software
  - software-guide
  - golang
  - golang-guide
---

See [github.com/hoani/fibonacci](https://github.com/hoani/fibonacci) for code in this example.

## Related guides

* [Protobuf basics](https://hoani.net/posts/guides/2021-10-30-protobufBasics/)

## Installation

```
go install google.golang.org/protobuf/cmd/protoc-gen-go@v1.26
go install google.golang.org/grpc/cmd/protoc-gen-go-grpc@v1.1
```

or see [gRPC quickstart prerequisites](https://grpc.io/docs/languages/go/quickstart/#prerequisites)

## GRPC Example

In this example, we will use a GRPC server to calculate fibonacci numbers:

```java
// fibonacci.proto

syntax = "proto3";

package fibonacci;
option go_package = "github.com/hoani/fibonacci";

service Fibonacci {
    rpc AtIndex(Number) returns (Number);
    rpc GetSequence(Number) returns (stream Number);
    rpc SumIndicies(stream Number) returns (Number);
    rpc StreamSequence(stream Number) returns (stream Number);
}

message Number {
    int32 value = 1;
}
```

Each rpc method represents a different call type:
* Unary 
    * single call single response
* Server Stream 
    * single call, stream response
* Client Stream
    * stream call, single response
* Bidirectional stream
    * stream call, stream response

## Setup

Make your root folder a module:
```sh
go mod init github.com/hoani/fibonacci
go mod tidy
```

Compile `fibonacci.proto` for go:

```sh
protoc --go_out=$OUT_PATH --go_opt=paths=source_relative \
       --go-grpc_out=$OUT_PATH --go-grpc_opt=paths=source_relative \
       $SRC_PATH/fibonacci.proto
```

Because the `go_package` in `fibonacci.proto` is `"github.com/hoani/fibonacci/proto`, I set `OUT_PATH` to `./proto`.

## Unary RPC Call

First we will just handle the unary call:

```java
    rpc AtIndex(Number) returns (Number);
```

### Server

The server takes GRPC calls and is assinged a port.

```golang
// server.go
package main

import (
    "context"
    "log"
    "net"

    fibonacci "github.com/hoani/fibonacci/proto"
    "google.golang.org/grpc"
)

func calculate(index int32) int32 {
    var a, b int32 = 0, 1
    for i := 0; i < int(index); i++ {
        a, b = b, a+b
    }
    return a
}

type server struct {
    fibonacci.UnimplementedFibonacciServer
}

func (s *server) AtIndex(
    ctx context.Context,
    in *fibonacci.Number,
) (*fibonacci.Number, error) {
    return &fibonacci.Number{Value: calculate(in.Value)}, nil
}

func main() {
    l, err := net.Listen("tcp", ":1337")
    if err != nil {
        log.Fatalf("failed to listen: %v", err)
    }
    s := grpc.NewServer()
    fibonacci.RegisterFibonacciServer(s, &server{})
    log.Printf("server listening at %v", l.Addr())
    if err := s.Serve(l); err != nil {
        log.Fatalf("failed to serve: %v", err)
    }
}
```

### Client

Now we create a client which makes a unary call to the server.

```go
// client_unary.go
package main

import (
    "context"
    "fmt"
    "log"
    "strconv"
    "time"

    fibonacci "github.com/hoani/fibonacci/proto"
    "google.golang.org/grpc"
)

func getUserInteger() int {
    var indexStr string
    fmt.Print("enter an index integer: ")
    fmt.Scanln(&indexStr)
    index, err := strconv.Atoi(indexStr)
    if err != nil {
        log.Fatalf("entered invalid integer `%v`", indexStr)
    }
    return index
}

func main() {
    conn, err := grpc.Dial("localhost:1337",
        grpc.WithInsecure(),
        grpc.WithBlock())
    if err != nil {
        log.Fatalf("did not connect: %v", err)
    }
    defer conn.Close()
    c := fibonacci.NewFibonacciClient(conn)

    index := getUserInteger()

    ctx, cancel := context.WithTimeout(
        context.Background(), 
        time.Second,
    )
    defer cancel()

    request := &fibonacci.Number{Value: int32(index)}
    r, err := c.AtIndex(ctx, request)
    if err != nil {
        log.Fatalf("call failed: %v", err)
    }
    log.Printf("Fibonacci #%v: %v", index, r.Value)
}
```

Running the server and client, we get:

```sh
$ go run server.go
2021/10/31 16:55:35 server listening at [::]:1337
```

```sh
$ go run ./client_unary.go 
enter an index integer: 10
2021/10/31 16:57:30 Fibonacci #10: 55
```

## Server Stream RPC

Next, we will implement the server stream RPC:

```java
    rpc GetSequence(Number) returns (stream Number);
```

### Server 

Add the following `server` method to `server.go`:

```go
func (s *server) GetSequence(
    in *fibonacci.Number, 
    stream fibonacci.Fibonacci_GetSequenceServer,
) error {
    var i int32 = 0
    for i = 0; i < in.Value; i++ {
        result := &fibonacci.Number{Value: calculate(i+1)}
        if err := stream.Send(result); err != nil {
            return err
        }
    }
    return nil
}
```

### Client

We can create a client which calls `GetSequence`

```go
// client_server_stream.go
package main

import (
    "context"
    "fmt"
    "io"
    "log"
    "strconv"

    fibonacci "github.com/hoani/fibonacci/proto"
    "google.golang.org/grpc"
)

func getUserInteger() int {
    var indexStr string
    fmt.Print("enter an index integer: ")
    fmt.Scanln(&indexStr)
    index, err := strconv.Atoi(indexStr)
    if err != nil {
        log.Fatalf("entered invalid integer `%v`", indexStr)
    }
    return index
}

func main() {
    conn, err := grpc.Dial("localhost:1337",
        grpc.WithInsecure(),
        grpc.WithBlock())
    if err != nil {
        log.Fatalf("did not connect: %v", err)
    }
    defer conn.Close()
    c := fibonacci.NewFibonacciClient(conn)

    index := getUserInteger()

    stream, err := c.GetSequence(
        context.Background(),
        &fibonacci.Number{Value: int32(index)},
    )
    if err != nil {
        log.Fatalf("call failed: %v", err)
    }
    i := 0
    for {
        r, err := stream.Recv()
        if err == io.EOF {
            break
        }
        if err != nil {
            log.Fatalf("%v.ListFeatures(_) = _, %v", c, err)
        }
        i += 1
        log.Printf("Fibonacci #%v: %v", i, r.Value)
    }
}
```

Running the server and client, we get:

```sh
$ go run server.go
2021/10/31 16:55:35 server listening at [::]:1337
```

```
$ go run ./client_server_stream.go 
enter an index integer: 8
2021/10/31 17:41:19 Fibonacci #1: 1
2021/10/31 17:41:19 Fibonacci #2: 1
2021/10/31 17:41:19 Fibonacci #3: 2
2021/10/31 17:41:19 Fibonacci #4: 3
2021/10/31 17:41:19 Fibonacci #5: 5
2021/10/31 17:41:19 Fibonacci #6: 8
2021/10/31 17:41:19 Fibonacci #7: 13
2021/10/31 17:41:19 Fibonacci #8: 21
```

## Client Stream RPC

Next, we will implement the client stream RPC:

```java
    rpc SumIndicies(stream Number) returns (Number);
```

This service takes a series of indices from the client, and sums the corresponding Fibonacci values.

### Server

Add the following `server` method to `server.go`:

```go
func (s *server) SumIndicies(
    stream fibonacci.Fibonacci_SumIndiciesServer,
) error {
    result := &fibonacci.Number{Value: 0}
    for {
        request, err := stream.Recv()
        if err == io.EOF {
            return stream.SendAndClose(result)
        }
        if err != nil {
            return err
        }
        result.Value += calculate(request.Value)
    }
}
```

### Client

We can create a client which calls `SumIndices`

```go
// client_client_stream.go
package main

import (
    "context"
    "fmt"
    "log"
    "strconv"

    fibonacci "github.com/hoani/fibonacci/proto"
    "google.golang.org/grpc"
)

func getUserInteger() (int, error) {
    var indexStr string
    fmt.Print("enter an index (or nothing to stop): ")
    fmt.Scanln(&indexStr)
    return strconv.Atoi(indexStr)
}

func main() {
    conn, err := grpc.Dial("localhost:1337",
        grpc.WithInsecure(),
        grpc.WithBlock())
    if err != nil {
        log.Fatalf("did not connect: %v", err)
    }
    defer conn.Close()
    c := fibonacci.NewFibonacciClient(conn)

    stream, err := c.SumIndicies(context.Background())
    if err != nil {
        log.Fatalf("call failed: %v", err)
    }

    for {
        index, err := getUserInteger()
        if err != nil {
            r, err := stream.CloseAndRecv()
            if err != nil {
                log.Fatalf("error closing %v", err)
            }
            log.Printf("Sum: %v", r.Value)
            break
        }
        increment := &fibonacci.Number{Value: int32(index)}
        err = stream.Send(increment)
        if err != nil {
            log.Fatalf("error sending %v", err)
        }
    }
}
```

Running the server and client, we get:

```sh
$ go run server.go
2021/10/31 16:55:35 server listening at [::]:1337
```

```sh
$ go run ./client_client_stream.go 
enter an index (or nothing to stop): 4
enter an index (or nothing to stop): 7
enter an index (or nothing to stop): 3
enter an index (or nothing to stop): 
2021/10/31 17:59:43 Sum: 18
```

## Bidirectional Stream RPC 

Finally, we will implement the bidirectional stream RPC:

```java
    rpc StreamSequence(stream Number) returns (stream Number);
```

This service takes a series of indices from the client, and sums the corresponding Fibonacci values.

### Server

Add the following `server` method to `server.go`:

```go
func (s *server) StreamSequence(
    stream fibonacci.Fibonacci_StreamSequenceServer,
) error {
    var index int32 = 0
    for {
        request, err := stream.Recv()
        if err == io.EOF {
            return nil
        }
        if err != nil {
            return err
        }

        for i := 0; i < int(request.Value); i++ {
            index++
            result := &fibonacci.Number{
                Value: calculate(index),
            }
            if err := stream.Send(result); err != nil {
                return err
            }
        }
    }
}
```

### Client

We can create a client which calls `StreamSequence`:

```go
// client_bidirectional_stream.go
package main

import (
    "context"
    "fmt"
    "io"
    "log"
    "strconv"
    "time"

    fibonacci "github.com/hoani/fibonacci/proto"
    "google.golang.org/grpc"
)

func getUserInteger() (int, error) {
    var indexStr string
    fmt.Print("enter the next index (or nothing to stop): ")
    fmt.Scanln(&indexStr)
    return strconv.Atoi(indexStr)
}

func main() {
    conn, err := grpc.Dial("localhost:1337",
        grpc.WithInsecure(),
        grpc.WithBlock())
    if err != nil {
        log.Fatalf("did not connect: %v", err)
    }
    defer conn.Close()
    c := fibonacci.NewFibonacciClient(conn)

    stream, err := c.StreamSequence(context.Background())
    if err != nil {
        log.Fatalf("call failed: %v", err)
    }

    rxDone := make(chan struct{})
    go func() {
        index := 0
        for {
            r, err := stream.Recv()
            if err == io.EOF {
                close(rxDone)
                return
            }
            if err != nil {
                log.Fatalf("Receive error: %v", err)
            }
            index += 1
            log.Printf("Fibonacci #%v: %v", index, r.Value)
        }
    }()
    for {
        increment, err := getUserInteger()
        if err != nil {
            err := stream.CloseSend()
            if err != nil {
                log.Fatalf("error closing %v", err)
            }
            break
        }
        request := &fibonacci.Number{ Value: int32(increment) }
        if err := stream.Send(request); err != nil {
            log.Fatalf("send failed: %v", err)
        }
        <-time.After(time.Second)
    }
    <-rxDone
}
```

Running the server and client, we get:

```sh
$ go run server.go
2021/10/31 16:55:35 server listening at [::]:1337
```

```sh
$ go run ./client_bidirectional_stream.go 
enter the next index (or nothing to stop): 4
2021/10/31 18:19:23 Fibonacci #1: 1
2021/10/31 18:19:23 Fibonacci #2: 1
2021/10/31 18:19:23 Fibonacci #3: 2
2021/10/31 18:19:23 Fibonacci #4: 3
enter the next index (or nothing to stop): 3
2021/10/31 18:19:25 Fibonacci #5: 5
2021/10/31 18:19:25 Fibonacci #6: 8
2021/10/31 18:19:25 Fibonacci #7: 13
enter the next index (or nothing to stop): 
```
