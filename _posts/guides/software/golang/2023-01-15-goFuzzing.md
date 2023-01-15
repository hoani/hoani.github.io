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

{% include video id="lB8UJJp6UrU" provider="youtube" %}

# Fuzz test strucutre

Fuzz tests look like:

```go
func Fuzz_MyTest(f *testing.F) {
  f.Add([]byte{1, 2, 3})

  f.Fuzz(func(t *testing.T, a []byte){
    // Test logic applied to the input.
  }
}
```

`Fuzz_MyTest(f *testing.F)` declares the fuzz test.

`f.Add(...)` specifies a "seed corpus". This is used as a starting point for the fuzzer to generate randomized inputs. We can have more than one `f.Add(...)` if we want to make sure specific cases are covered.

`f.Fuzz(func(t *testing.T, ...)` speficies the actual test to run on the randomized inputs.

In this case I have specified `a []byte` as the input argument. However, fuzzing arguments can be one or more of any of the following types:
```go
int, int8, int16, int32, int64, 
uint, uint8, uint16, uint32, uint64,
rune, byte, []byte, string,
float32, float64, bool 
```

# Running fuzz tests

We can call: 
```
go test .
```
This will run our fuzz test with the "seed corpuses" (and generated corpuses if those exist), but no randomized tests will be run.

To run the fuzzer, run:
```
go test -fuzz=.
```
This will run until a failure occurs. If it is impossible to make the test fail or panic, this command will run forever.

If we want the fuzzer to run for a specific amount of time (i.e for CI), then run:
```
go test -fuzz=. -fuzztime=5s
```
This will run the fuzzer for 5 seconds only.

# Generated corpus

When a fuzz test fails, it will generate something called a "generated corpus", which is a specification of the exact inputs which caused the failure.

This will be stored in a `testdata/fuzz/<test name>/<hash>` file which will look something like:
```
go test fuzz v1
[]byte("")
```

Where the second line specifies the inputs.

You may consider commiting the generated corpus to your project, because it provides regression for your test - that is, it will be run alongside your "seed corpus" whenever your fuzz test is run with `go test`.
