---
title: "Rust Lifetime"
excerpt: "Using Lifetimes in Rust"
classes: wide
permalink: /guides/software/rust/lifetimes
categories:
  - guide
  - rust
  - rust-guide
---

Lifetimes are defined using the `'` symbol. They ensure that references remain valid for the scope they are used in.

For example, we would like to know that the reference created by this function will live long enough to be used in the calling block:
```rust
fn longest_name(name_1: &str, name_2: &str) -> &str {}
```

Rust automatically assigns lifetimes, but will need help determining which lifetime to apply to the return type:
```rust
fn longest_name<'a, 'b>(name_1: &'a str, name_2: &'b str) -> &str {}
```

We can define lifetimes explicitly - in this case make all references share the same lifetime guarantee:
```rust
fn longest_name<'a>(name_1: &'a str, name_2: &'a str) -> &'a str {}
```

If the input references are not within the same lifetimes of eachother, we can ensure `'b` is at least within the same lifetime as `'a` by using:
```rust
fn longest_name<'a, 'b: 'a>(name_1: &'a str, name_2: &'b str) -> &'a str {}
```

### Static lifetimes
`'static` is a special lifetime which will last the entirety of the program.

All `const` values are `'static` by default. Variables can also be static.

We may choose to require only `'static` outputs to a function:
```rust
fn version_info() -> &'static str {}
```

### Lifetimes in structs

Structs which reference memory also require lifetime definitions.
```rust
struct ImuData<'a> {
	accel_resolution: f32,
	accel_raw: &'a Vec<u32>,
}
```

### External Links:
* [Doug Milford - Rust Lifetimes](https://www.youtube.com/watch?v=1QoT9fmPYr8)
