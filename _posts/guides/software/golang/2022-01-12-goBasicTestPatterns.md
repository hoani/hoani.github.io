---
title: "Go Basic Test Patterns"
excerpt: "Practical testing in go"
permalink: "/guides/software/golang/test-patterns"
toc: true
categories:
  - guide
  - golang
  - software
---

This guide provides some useful patterns for writing tests. For the basic go testing cheatsheet see [Go Unit Testing](../2020-12-01-golangUnitTests)

## File Organization

Packages should have tests to prove their interface to other packages. For example:

```
/scheduler
    scheduler.go
    time-converter.go
```

Should include at least a file `scheduler_test.go` which tests the package interface.

The file `scheduler_test.go` should be part of `package scheduler`, and not `package scheduler_test`. This is because `go test -cover` has a harder time including tests from external packages as part of coverage metrics.

## Test Functions

Test functions have the following anatomy:
```go
func Test_Scheduler_ExecutesCallbackOnTime(t *testing.T) {
    //...do stuff
}
```

The vital organs of the test function are:
* `func Test...` which marks the function as a test function
* `(t *testing.T)` which is the signature of a test function
* Any failure mechanism in the body of the test which indicate the test has failed

When naming tests, consider:
* Include the package name if the package is part of a larger project
* Include what requirement is being tested

## Making a test fail

Use `t.Fatal("...")` or `t.Fatalf("...", ...)` to fail tests.

Don't use:
* `t.Error("...")` allows multiple failures to occur which pollutes the test output making analysis harder.
* `panic("...")` or `log.Fatal("...")` 
    * we are writing use the test framework

## Testify and External Test Packages

The only thing better than `t.Fatal` is `github.com/stretchr/testify/assert`.

The syntax is expressive without polluting tests with conditional logic:
```go
msg := cache.Pop()
assert.NotContains(t, cache.Data(), msg)

b, err := json.Marshal(msg)
assert.Nil(t, err)
assert.Equal(t, string(b), `{"dest":"a:1337","p":"test"}`)
```

Output looks like:
```
=== RUN Test_Scheduler_ExecutesCallbackOnTime
    scheduler_test.go:52:
                Error Trace:    scheduler_test.go:52
                Error:          Not equal:
                                expected: 2000
                                actual  : 123
                Test:           Test_Scheduler_ExecutesCallbackOnTime
--- FAIL: Test_Scheduler_ExecutesCallbackOnTime
```

Which is consistent and easy to analyze.

Don't use the package `github.com/stretchr/testify/require`, like `t.Error` it pollutes the test output.

## Test Logging

Avoid polluting test output with logging as much as possible.

`t.Log` is available as a debugging tool when necessary, but I avoid merging test code with `t.Log` into main.

## Subtests

Subtests are great when we have constant data which is common among test cases, or alternatively we have a very long stateful test with stages.

```go
func Test_FetchUserInformation(t *testing.T) {
    s := FakeServer()
    s.Users := []User{
        {Id: 1, Name: "Ed"}
    }

    client := NewClient()

    t.Run("dial server", func(t *testing.T){
        err := client.Dial(s.Addr)
        assert.Nil(t, err)
    })

    t.Run("fetch user", func(t *testing.T){
        user := client.FetchUser(1)
        assert.Equal(t, "Ed", user.Name)
    })
}
```

## Table driven tests

Table driven tests are popular in go. Combined with `testify/assert` they are very expressive:

```go
func Test_FetchUserInformation(t *testing.T) {
    s := FakeServer()
    s.Users := []User{
        {Id: 1, Name: "Ed"}
        {Id: 12, Name: "Stan"}
    }

    testCases := struct{
        name string
        userId int
        expected *User
    } {
        {
            name: "get Ed",
            userId: 1,
            expected: &User{Id: 1, Name: "Ed"},
        },
        {
            name: "get Stan",
            userId: 12,
            expected: &User{Id: 12, Name: "Stan"},
        }
        {
            name: "unknown User",
            userId: 123,
            expected: nil,
        }
    }

    for _, tc := range testCases {
        t.Run(tc.name, func(t *testing.T){
            client := NewClient()
            err := client.Dial(s.Addr)
            assert.Nil(t, err)

            user := client.FetchUser(tc.userId)

            if tc.expected == nil {
                assert.Nil(t, user)
            } else {
                assert.NotNil(t, user)
                assert.Equal(t, "Ed", user.Name)
            }
        })
    }
}
```

