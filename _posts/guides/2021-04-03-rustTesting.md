---
title: "Rust Testing"
excerpt: "Cheatsheet for Rust testing syntax"
toc: true
categories:
  - guide
  - rust
  - rust-guide
---

## Unit Testing

Unit test code is in the same file as the source under a test module

Say we had the following code in a file called `line.rs`

```rust
pub fn line(x : f64, m : f64, c : f64) -> f64 {
  x * m + c
}

#[cfg(test)]
mod tests {
  use super::*;

  #[test]
  fn test_line() {
    assert_eq!(line(0.0, 1.0, 2.0), 2.0);
    assert_eq!(line(1.0, 1.0, 2.0), 3.0);
    assert_eq!(line(2.0, 1.0, 2.0), 4.0);
    assert_eq!(line(-1.0, 1.0, 2.0), 1.0);
  }
}
```
`use super::*` imports everything from the parent module

We can compile and run this with:
```sh
rustc line.rs --test
./line
```

To give the binary a different name:
```sh
rustc --crate-name line_test line.rs --test
./line_test 
```

### Test Asserts

```rs
assert!(line(1.0, 1.0, 2.0) == 3.0);
assert_eq!(line(1.0, 1.0, 2.0), 3.0);
assert_ne!(line(1.0, 1.0, 2.0), 4.0);
```

A trickier case is when we expect our code to panic:

```rs
#[test]
#[should_panic]
fn test_that_panics() {
  function_that_panics();
}

#[test]
#[should_panic(expected = "Oh no!)"]
fn test_that_panics() {
  panic!("Oh no!");
}
```

Result-based asserts:
```rs
#[test]
fn test_with_result() -> Result<(), String> {
  if line(1.0, 1.0, 2.0) == 3.0 {
    Ok(())
  } else {
    Err(String::from("Failed"))
  }
}
```

### Ignoring tests

We can ignore tests using:
```rs
#[test]
#[ignore]
fn test_with_flakey_code() {
  // ...
```

If we want to check if ignored tests are working, run the binary with the ignored flag.

```sh
./line_test --ignored
```

Or run `cargo test`, but pass `--ignored` to the binary:

```sh
cargo test -- --ignored
```

## Integration Testing

A suite of integration tests can be defined per crate. They are located in the `tests` directory right next to the `src` directory.

Integration tests only make use of a crate's public interface.

To integration test `line()` in a package called `line`:

```rs
use line;

#[test]
fn compute_line() {
    assert_eq!(line::line(1.5,1.0,-1.5), 0.0)
}
```

Each file in the `tests` directory is compiled as it's own crate. This means scope cannot be shared between integration test files.

Integration tests are run the same way as unit tests.

## Running Tests with Cargo

Cargo attempts to run unit, integration and doc tests.

In a cargo project, run:
```sh
cargo test
```

To filter only tests which contain `line`:
```sh
cargo test line
```

If `line` is part of a workspace package, run:
```sh
cargo test --workspace
```
This runs tests for all packages specified in the [workspace](https://doc.rust-lang.org/cargo/reference/workspaces.html).

