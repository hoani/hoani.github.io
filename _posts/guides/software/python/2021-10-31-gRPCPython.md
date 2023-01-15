---
title: "Python gRPC"
excerpt: "Basic gRPC using Python"
toc: true
permalink: /guides/software/python/grpc
categories:
  - guide
  - python
  - python-guide
---

See [github.com/hoani/fibonacci](https://github.com/hoani/fibonacci) for code in this example.

## Related guides

* [Protobuf basics](/guides/software/protobuf)

## Installation

```
python3 -m pip install grpcio grpcio-tools
```

## GRPC Example

In this example, we will use a GRPC server to calculate fibonacci numbers:

```java
// fibonacci.proto

syntax = "proto3";

package fibonacci;

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

## Compile

```
python -m grpc_tools.protoc -I=$SRC_PATH \
  --python_out=$OUT_PATH --grpc_python_out=$OUT_PATH \
  fibonacci.proto
```

## Unary RPC Call

First we will just handle the unary call:

```java
    rpc AtIndex(Number) returns (Number);
```

### Server

The server takes GRPC calls and is assinged a port.

```py
# server.py
from concurrent import futures
import logging

import grpc
import fibonacci_pb2
import fibonacci_pb2_grpc


class Fibonacci(fibonacci_pb2_grpc.FibonacciServicer):

    def calculate(self, index):
        a, b = 0, 1
        for _ in range(index):
            a, b = b, a + b
        return a

    def AtIndex(self, request : fibonacci_pb2.Number, context):
        result = self.calculate(request.value)
        return fibonacci_pb2.Number(value=result)


def serve():
    server = grpc.server(
        futures.ThreadPoolExecutor(max_workers=10)
    )
    fibonacci_pb2_grpc.add_FibonacciServicer_to_server(
        Fibonacci(), server
    )
    server.add_insecure_port('[::]:1337')
    server.start()
    server.wait_for_termination()


if __name__ == '__main__':
    logging.basicConfig()
    serve()
```

### Client

Now we create a client which makes a unary call to the server.

```py
# client_unary.py
import logging

import grpc
import fibonacci_pb2
import fibonacci_pb2_grpc

if __name__ == '__main__':
    logging.basicConfig()

    with grpc.insecure_channel('localhost:1337') as channel:
        stub = fibonacci_pb2_grpc.FibonacciStub(channel)
        value = int(input("enter an index integer: "))
        response = stub.AtIndex(fibonacci_pb2.Number(value=value))
    print(f"Fibonacci #{value}: {response.value}")
```

Running the server and client, we get:

```sh
$ python server.py
```

```sh
$ python client_unary.py
enter an index integer: 7
Fibonacci #7: 13
```

## Server Stream RPC

Next, we will implement the server stream RPC:

```java
    rpc GetSequence(Number) returns (stream Number);
```

### Server 

Add the following method to the `Fibonacci` class in `server.py`:

```py
def GetSequence(self, request, context):
    for index in range(request.value):
        result = self.calculate(index + 1)
        yield fibonacci_pb2.Number(value=result)
```

### Client

We can create a client which calls `GetSequence`

```py
# client_server_stream.py
import grpc
import fibonacci_pb2
import fibonacci_pb2_grpc

if __name__ == '__main__':
    with grpc.insecure_channel('localhost:1337') as channel:
        stub = fibonacci_pb2_grpc.FibonacciStub(channel)
        value = int(input("enter an index integer: "))
        request = fibonacci_pb2.Number(value=value)
        responses = stub.GetSequence(request)
        index = 0
        for response in responses:
            index += 1
            print(f"Fibonacci #{index}: {response.value}")
```

Running the server and client, we get:

```sh
$ python server.py
```

```
$ python client_server_stream.py 
enter an index integer: 7
Fibonacci #1: 1
Fibonacci #2: 1
Fibonacci #3: 2
Fibonacci #4: 3
Fibonacci #5: 5
Fibonacci #6: 8
Fibonacci #7: 13
```

## Client Stream RPC

Next, we will implement the client stream RPC:

```java
    rpc SumIndicies(stream Number) returns (Number);
```

This service takes a series of indices from the client, and sums the corresponding Fibonacci values.

### Server

Add the following method to the `Fibonacci` class in `server.py`:

```py
def SumIndicies(self, requests, context):
    result = 0
    for request in requests:
        result += self.calculate(request.value)
    return fibonacci_pb2.Number(value=result)
```

### Client

We can create a client which calls `SumIndices`

```py
# client_client_stream.py
import grpc
import fibonacci_pb2
import fibonacci_pb2_grpc

def sendIndicies():
    try:
        while(True):
            valstr = input("enter an index (or nothing to stop): ")
            yield fibonacci_pb2.Number(value=int(valstr))
    except:
        print("done sending")

if __name__ == '__main__':
    with grpc.insecure_channel('localhost:1337') as channel:
        stub = fibonacci_pb2_grpc.FibonacciStub(channel)
        response = stub.SumIndicies(sendIndicies())
    print(f"Sum: {response.value}")
```

Running the server and client, we get:

```sh
$ python server.py
```

```sh
$ python client_client_stream.py 
enter an index (or nothing to stop): 7
enter an index (or nothing to stop): 5
enter an index (or nothing to stop): 
done sending
Sum: 18
```

## Bidirectional Stream RPC 

Finally, we will implement the bidirectional stream RPC:

```java
    rpc StreamSequence(stream Number) returns (stream Number);
```

This service takes a series of indices from the client, and sums the corresponding Fibonacci values.

### Server

Add the following method to the `Fibonacci` class in `server.py`:

```py
def StreamSequence(self, requests, context):
    index = 0
    for request in requests:
        for _ in range(request.value):
            index += 1
            result = self.calculate(index)
            yield fibonacci_pb2.Number(value=result)
```

### Client

We can create a client which calls `StreamSequence`:

```py
# client_bidirectional_stream.py
import grpc
from time import sleep
import fibonacci_pb2
import fibonacci_pb2_grpc

def nextIncrement():
    try:
        while(True):
            sleep(1)
            value = int(input("enter the next increment "+
                "(or nothing to stop): "))
            yield fibonacci_pb2.Number(value=value)
    except:
        print("done sending")

if __name__ == '__main__':
    with grpc.insecure_channel('localhost:1337') as channel:
        stub = fibonacci_pb2_grpc.FibonacciStub(channel)
        responses = stub.StreamSequence(nextIncrement())
        index = 0
        for response in responses:
            index += 1
            print(f"Fibonacci #{index}: {response.value}")
```

Running the server and client, we get:

```sh
$ python server.py
```

```sh
$ python client_bidirectional_stream.py 
enter the next increment (or nothing to stop): 4
Fibonacci #1: 1
Fibonacci #2: 1
Fibonacci #3: 2
Fibonacci #4: 3
enter the next increment (or nothing to stop): 2
Fibonacci #5: 5
Fibonacci #6: 8
enter the next increment (or nothing to stop): 1
Fibonacci #7: 13
enter the next increment (or nothing to stop): 
done sending
```

