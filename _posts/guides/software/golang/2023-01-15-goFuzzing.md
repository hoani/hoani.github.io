---
title: "Go Fuzz Testing"
excerpt: "Use Go's inbuilt fuzz testing tools."
toc: true
permalink: "/guides/software/golang/fuzzing"
categories:
  - guide
  - golang
  - golang-guide
---

SWIG allows us to integrate C++ code and libraries into Go.

The underlying technology is still [cgo](/guides/golang/cgo), and using SWIG has all of the usual [drawbacks](/guides/golang/cgo#notes-on-cgo) such as limited debugging and cross-compilation suppor.

## Installation

On MacOS:
```
brew install swig
```

On Debian/Ubuntu:
```
apt-get install swig3.0
ln -s /usr/bin/swig3.0 /usr/bin/swig
```

On Windows, follow the [openbox instructions](https://open-box.readthedocs.io/en/latest/installation/install_swig.html#windows) to download and install on your machine.

## Creating a SWIG package

Say we have two simple cpp files in a folder named `adder`:

```
|- adder/
 |- adder.cpp
 |- adder.hpp
```

`adder.hpp`:
```cpp
#ifndef _ADDER_H_
#define _ADDER_H_

class Adder
{
public:
  Adder(): _v(0) {};
  void Add(int v);
  int Get();
private:
  int _v;
};

#endif // _ADDER_H_
```

`adder.cpp`:
```cpp
#include "adder.hpp"

void Adder::Add(int v) { _v += v; }
int Adder::Get() { return _v; }
```

We would like to convert this `adder` folder into a SWIG package. To do this we add two files:
* `adder.swigcxx`
* `package.go`

We now have:

```
|- adder/
 |- adder.cpp
 |- adder.hpp
 |- adder.swigcxx
 |- package.go
```

For a simple example like this, `package.go` just includes the cgo import:
```go
package adder

import "C"
```

The `adder.swigcxx` file includes `adder.h`:
```
%module adder
%{
#include "adder.hpp"
%}

%include "adder.hpp"
```

## Using our adder package

If we add a `main.go` file and call `go mod init tut/swig` in our root directory, we will get:
```
|- adder/
 |- adder.cpp
 |- adder.hpp
 |- adder.swigcxx
 |- package.go
|- main.go
|- go.mod
```

Inside `main.go`, we can call `adder`:
```go
package main

import (
	"fmt"
	"tut/swig/adder"
)

func main() {
	a := adder.NewAdder()
	defer adder.DeleteAdder(a)
	a.Add(1)
	a.Add(2)
	fmt.Printf("value %d\n", a.Get())
}
```

Calling go run will compile the `adder` C++ code and we get:
```
└─ $ ▶ go run .
value: 3
```

## Memory safety

In our `main()` function, we created our `Adder` class using:
```go
a := adder.NewAdder()
```

This allocates memory which Go's garbage collector does not manage. So we need to make sure we delete the `Adder` object when we are done:
```go
defer adder.DeleteAdder(a)
```

Further information on this can be found in [SWIG and Go: Go Class Memory Management](https://www.swig.org/Doc3.0/Go.html#Go_class_memory)

## Analyzing SWIG generated go code

When we call `go build` or `go run` `swig` is invoked. We can check this with the `-x` flag:

```
└─ $ ▶ go clean --cache && go build -x
WORK=/var/folders/z_/cgpk1xl17yq02dg3cp_wgvk00000gp/T/go-build2159719876
mkdir -p $WORK/b040/
...
cd /Users/hoani/go/src/sandbox/cppswig/adder
swig -go -cgo -intgosize 64 -module adder -o $WORK/b040/adder_wrap.cxx -outdir $WORK/b040/ -c++ adder.swigcxx
```

If we want to see the generated code, we can compile with the `-work` flag and navigate to the work directory:
```
└─ $ ▶ go clean --cache && go build -x --work                                  
WORK=/var/folders/z_/cgpk1xl17yq02dg3cp_wgvk00000gp/T/go-build578608493
...
└─ $ ▶ code /var/folders/z_/cgpk1xl17yq02dg3cp_wgvk00000gp/T/go-build578608493
```

In my case, the file `b040/_adder_swig.go` contained all the wrapper defintions for the `Adder` class.

<figure>
    <img src="/assets/images/posts/guides/go/swig/000_adder.png">
</figure>

## Advanced Go w/ SWIG

### Linking a Shared Object Library

If we have a shared object library `libMyLib.so` and its corresponding header files our SWIG package may look like:
```
|- mylib/
 |- libMyLib.so
 |- mylib.h
 |- mylib.swigcxx
 |- package.go
```

Inside `package.go` we would add some `LDFLAGS`:
```go
package mylib

// #cgo LDFLAGS: -L${SRCDIR} -lMyLib
import "C"
```

This should compile, but it is important that `libMyLib.so` is also copied to a location on your machine (such as `/usr/local/lib`) where it can be discovered when executing your go binary.

Alternatively, if you know that the library will always be provided alongside your binary, for example:
```
|-bin/
 |-myProgram
|-lib/
 |-libMyLib.so
```
You can specify additional locations for your binary to find your libraries in `package.go`:
```go
package myLib

// #cgo CXXFLAGS: -std=c++11
// #cgo LDFLAGS: -L${SRCDIR} -Wl,-rpath=\$ORIGIN/../lib -lMyLib
import "C"
```

### Std String

SWIG will compile the cpp `std::string` to `<pkgname>.Std_string`. But ideally, we would like to treat `std::string` as a Go `string`.

For example, in the `adder` example, if we want to add a `Print` method to our adder class in `adder.hpp`:
```cpp
...
#include <string>
...
class Adder
...
  void Print(std::string prefix);
```

And we add the implementation in `adder.cpp`:
```cpp
...
#include <iostream>
...
void Adder::Print(std::string prefix) {
  std::cout << prefix << ": " << _v << std::endl;
}
```

And we call this in our `main` function:
```go
  a.Print("adder value")
```

We get the compile error:
```
└─ $ ▶ go run .
# tut/swig
./main.go:12:10: cannot use "adder value" (constant of type string) as type adder.Std_string in argument to a.Print:
        string does not implement adder.Std_string (missing Swigcptr method)
```

To fix this, SWIG provides a `std_string` inport we can use in our `adder.swigcxx` file:

```
%module adder
%{
#include "adder.hpp"
%}

%include "std_string.i"
%include "adder.hpp"
```

Now if we go run, it works:
```
└─ $ ▶ go run .
adder value: 3
```

### Namespaces

If the included C++ code's namespaces get in the way, it can be helpful to use the `using` directive in your SWIG file:
```
%module mypackage
%{
#include "trex.hpp"
using namespace dinosaurs
%}

%include "trex.hpp"
using namespace dinosaurs;
```

### Templating 

C++ templates need to be explicitly declared to use in Go.

For example, what if we added the following function to our `Adder` class:
```cpp
void AddMany(std::vector<int> vs) {
    for (int v : vs) {
      _v += v;
    }
  }
```

We now need to be able to pass a `std::vector<int>` from Go; which is an implementation of the `std::vector<T>` template.

To do this, update the `adder.swigcxx`:

```
...
%include "std_vector.i"
namespace std {
   %template(vectorint) vector<int>;
};
```

Note: `vectorint` can be named whatever we like, but it will be the Go equivalent to `std::vector<int>`.

Next, we can use this type in our `main` function:

```go
func main() {
  a := adder.NewAdder()
	defer adder.DeleteAdder(a)
	vint := adder.NewVectorint()
	defer adder.DeleteVectorint(vint)
	vint.Add(10)
	vint.Add(5)
	a.AddMany(vint)
  fmt.Printf("adder value %d\n", a.Get())
}
```

And if we run this we get:
```
└─ $ ▶ go run .
adder value: 18
```

Note: you may get a `warning: range-based for loop is a C++11 extension`. To make this go away, update `package.go` to use C++11 or higher:
```go
package adder

// #cgo CXXFLAGS: -std=c++11
import "C"
```
