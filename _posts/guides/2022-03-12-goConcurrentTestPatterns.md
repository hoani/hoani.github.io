---
title: "Go Concurrent Test Patterns"
excerpt: "Concurrent testing in go"
toc: true
categories:
  - guide
  - golang
  - golang-guide
---

This guide provides some useful patterns for writing test for concurrent code. For the basic go testing cheatsheet see [Go Unit Testing](../2020-12-01-golangUnitTests)

## Out-of-order Results

Say we have a test:

```go
func Test_GetLatestUsers() {
  expected := {"Alice", "Bob", "Charlie"}
  
  srv := NewUserServer(
    httptest.NewServer(UserRouter()),
  )
  defer srv.Close()

  for _, user := range expected{
    go PostCreateUser(srv.Addr, user)
  }

  time.Sleep(time.Second)
  assert.Equal(t, expected, srv.Users)
}
```

There are a number of things wrong with the test; but let's start with the fundamental one - it will fail often.

There is no guarantees around the order in which the new `go` routines will be executed, so the `assert.Equal` will often be asserting on out of order lists.

A better approach is to anticipate that the users will be out of order, and check that the `srv.Users` list is equivalent:

```go
  //...
  time.Sleep(time.Second)
  assert.Len(t, srv.Users, len(expected))
  for _, user := range expected {
      assert.Contains(t, srv.Users, user)
  }
```

If you don't use [Stretchr's testify](https://github.com/stretchr/testify) you can always create your own helper for checking list equivalence:
```go
func Equivalent(t *testing.T, expected, result []string) {
  t.Helper()
  if (len(expected) != len(result)) {
    t.Fatal("length mismatch")
  }
  for _, exp := range expected {
    found := false
    for _, res := range result {
      if res == exp {
        found = true
        break
      }
    }
    if !found {
      t.Fatal("%v not found in %v", exp, result)
    }
  }
}
```

## Race conditions

Say we have a counter which increments each time our fake service is called:

```go
func Test_SetConfig() {
  sendCalled := 0
  
  s := NewConfigurator(
    &FakeConfiguratorService{
      onSend: func(k, v string) {
        sendCalled++
      }
    },
  )

  configs := map[string]string{
    "A": "a",
    "B": "b",
    "C": "c",
  }

  for k, v := range configs {
    go s.SendConfig(k, v)
  }

  time.Sleep(time.Second)
  assert.Equal(t, 3, sendCalled)
}
```

Every now and then, this test will fail. This is because of a data race on `sendCalled++`.

To detect the data race, you can run line:
```sh
go test . -race
```

And you should see something like:

```sh
==================
WARNING: DATA RACE
Read at 0x00c0003ba010 by goroutine 7:
# ...
Previous write at 0x00c0003ba010 by goroutine 9:
  my-test.Test_SetConfig.func1()
```

To fix the data race, use a mutex:

```go
func Test_SetConfig() {
  mu := sync.Mutex{}
  sendCalled := 0
  
  s := NewConfigurator(
    &FakeConfiguratorService{
      onSend: func(k, v string) {
        mu.Lock()
        defer mu.Unlock()
        sendCalled++
      }
    },
  )

  configs := map[string]string{
    "A": "a",
    "B": "b",
    "C": "c",
  }

  for k, v := range configs {
    go s.SendConfig(k, v)
  }

  time.Sleep(time.Second)

  mu.Lock()
  defer mu.Unlock()
  assert.Equal(t, 3, sendCalled)
}
```

Running `go test . -race` should no longer result in `DATA RACE` warnings.

## Remove arbitrary sleep

To really test that our concurrent tests won't randomly fail, one technique is to run it many times. For example:

```sh
go test . -run=Test_GetLatestUsers -count=1000
```

The problem with this is that we use `time.Sleep(time.Second)` to prevent the test from exiting before all of our go routines are done. This means the above all would take 17 minutes. We aren't guaranteed that one second is enough time either!

A better pattern is using wait groups. If you aren't familiar with `sync.WaitGroup`, it has three methods:
* `Add(n)` adds `n` to a `wait` counter
* `Done()` adds `1` to a `done` counter
* `Wait()` blocks until `done == wait` counter


For `Test_GetLatestUsers` the test will look something like:

```go
func Test_GetLatestUsers() {
  expected := {"Alice", "Bob", "Charlie"}

  wg := sync.Waitgroup{}
  wg.Add(len(expected))
  
  srv := NewUserServer(
    httptest.NewServer(UserRouter()),
  )
  defer srv.Close()

  for _, user := range expected{
    go func(){
      PostCreateUser(srv.Addr, user)
      wg.Done()
    }()
  }

  wg.Wait()
  assert.Len(t, srv.Users, len(expected))
  for _, user := range expected {
      assert.Contains(t, srv.Users, user)
  }
}
```

With this logic, the execution time of `1000` tests reduces from 1000 seconds to under 1 second.

### Testing for reasonable execution time

Adding the wait group above can allow a test to pass even when the go routines take much longer than expected to execute.

To add this timing expectation, we use a channel and `select` statement to detect failure.

```go
func Test_GetLatestUsers() {
  // ...

  for _, user := range expected{
    go func(){
      PostCreateUser(srv.Addr, user)
      wg.Done()
    }()
  }

  done := chan(struct{})
  go func() {
    wg.Wait()
    close(done)
  }()

  select{
    case <-done: // Expected
    case <-time.After(time.Second):
      t.Fatal("test timeout")
  }

  assert.Len(t, srv.Users, len(expected))
  for _, user := range expected {
      assert.Contains(t, srv.Users, user)
  }
}
```

This ensures that if a `go` routine takes too long, the longest the test will take to fail is only one second.

## Avoiding failures outside of the test routine

Test failures should not be triggered in new `go` routines. For example, don't do this:

```go
func Test_GetLatestUsers() {
  // ...

  for _, user := range expected{
    go func(){
      err := PostCreateUser(srv.Addr, user)
      if err != nil {
        t.Fatalf("error, %v", err)
      }
      wg.Done()
    }()
  }

  //...
}
```

If the test exits early (eg. timeout), but a child `go` routine calls `t.Fatalf`, it can cause a panic in the test suite.

Use an error channel, which we listen to on the select statement:

```go
func Test_GetLatestUsers() {
  // ...

  errCh := make(chan error, len(expected))

  for _, user := range expected{
    go func(){
      err := PostCreateUser(srv.Addr, user)
      if err != nil {
        errCh <- err
      }
      wg.Done()
    }()
  }

  //...
  select{
    case <-done: // Expected
    case err := <-errCh:
      t.Fatalf("error, %v", err)
    case <-time.After(time.Second):
      t.Fatal("test timeout")
  }

  //...
}
```


