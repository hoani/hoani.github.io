---
title: "Go Unit Testing"
excerpt: "Basic cheatsheet for golang"
toc: true
categories:
  - guide
  - golang
  - golang-guide
---

```golang
package PIDControl

import "testing"

func TestProportionalGain(t *testing.T) {
  regulator := PID(1.0, 0.0, 0.0)
  regulator.setpoint(1.0);

  regulator.update(0.1, 0.0); // time, state
  output := regulator.getOutput();
  
  if regulator.getOutput() != 1.0 {
    t.Errorf("P control failed, expected: %f, result: %f", 
             1.0, output)
  }
}
```

```sh
go test          # Run all tests in this package
go test -v       # Run all tests with verbose output
go test -run=Pro   # Run tests which contain `Pro`
go test -count=5 # Repeat tests 5 times
go test -count=1 # Does not use cached results
```

## Test Functions

Are declared with `Test` prefixed to the name and takes an argument of type `*testing.T` 
```golang
func TestDoSomething(t *testing.T) { //...
```

To make a test fail, we can use `Errorf` or `Fatalf`:
```golang
if expected != result {
  t.Errorf("Failed %f != %f", expected, result)
}
```
```golang
if expected != result {
  t.Fatalf("Failed %f != %f", expected, result)
}
```
* `Errorf` will continue to execute the remainder of the test body after failure
* `Fatalf` terminates the test case immediately on failure

## Integration-ish Tests

The `-short` flag can be used to only run essential unit tests for development. The tests which should be skipped need to be marked as follow:

```golang
func TestIntegration(t *testing.T) {
	if testing.Short() {
		t.Skip("skipping test in short mode.")
	}
	// Do lots of things
}
```

To skip `TestIntegration` call:
```sh
go test -short
```

## Setup/Teardown

To create a series of tests with common Setup/Teardown, use `Subtests`:
```golang
func TestMotorControl(t *testing.T) {
    // <setup code>
    t.Run("Forward", func(t *testing.T) { ... })
    t.Run("Backward", func(t *testing.T) { ... })
    // <tear-down code>
}
```

The effective test names of these subtests will be:
* `MotorControl/Forward`
* `MotorControl/Backward`

## Benchmarking

To check the performance of a function, can use Benchmarking:

```golang
func BenchmarkEncode(b *testing.B) {
    for i := 0; i < b.N; i++ {
        packet.Encode()
    }
}
```

To run all benchmarks:
```sh
go test -bench .
```
Otherwise, the input argument is a (loose) regex:
```sh
go test -bench Name
```