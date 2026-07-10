---
title: "Rust Basics"
excerpt: "Lists, Slices, Arrays, Strings, Loops, and Iterators"
toc: true
permalink: /guides/software/rust/basics
categories:
  - guide
  - rust
  - software
---

## Command line Basics

### Rustup

Compile `app.rs`

```sh
rustup app.rs
```

Show standard documentation

```sh
rustup doc --std
```

## Language Basics

### Arrays

Arrays are fixed in size.

```rs
let arr = [1, 2, 3, 4, 5]; // type: [i32; 5]
let size = arr.len();      // size: 5
let third = arr[2];        // val:  3
```

### Slices

A slice is a reference to an array (or part of an array).

```rs 
let arr = [1, 2, 3, 4, 5]; // type: [i32; 5]
sum(&arr);                 // &arr -> [1, 2, 3, 4, 5]
sum(&arr[0..2]);           // &arr[0..2] -> [1, 2]
```

A slice index is safely accessed with `get()`:

```rs
let slice = &arr;
let valid = slice.get(2);   // Some(3)
let invalid = slice.get(3); // None
```

### Vector

Vectors are dynamically allocated and live on the heap. 

We can add/remove elements when a vector is mutable:

```rs
let mut v = Vec::new();
v.push(10);
v.get(0); // Some(10)
v.get(2); // None
```

Vectors are coerced into slices the same way as arrays:

```rs
let slice = &v;
```

Vectors can be instantiated with an array-like declaration using `vec!`:

```rs 
let v = vec![10, 11, 12];
```

### Strings

Strings are Utf-8 encoded.

* `&str` represents string literals. It is a string slice
* `String` is a dynamically allocated string

```rs
let literal = "Hoani";       // &str
let s = literal.to_string(); // String
let l = &s;                  // &str
```

`format!()` is a convenient way to build strings.

```rs
let s = format!("{}, {}, {} \n", time, speed, voltage);
```

### Loops

```rs
for i in 0..3 {
    println!("Index {}", i);
}
```
Result:
```
Index 0
Index 1
Index 2
```

### Iterators

Iterators have a `next()` method which returns an `Option` containing `Some` until they are out of values (and then returns `None`).

```rs
let mut iter = 0..2;
println!("{:?}", iter.next());  // Some(0)
println!("{:?}", iter.next());  // Some(1)
println!("{:?}", iter.next());  // None
```

Arrays are converted into iterators using `arr.iter()`, slices can be used implicitly as an iterator.

### Structs

```rs
struct CommandLed {
  which: Led
  on: bool
}

impl CommandLed {
  fn decode(bytes: Vec<u8>) -> Self {
    // ...
  }
  fn encode(self) -> Vec<u8> {
    // ... Note: self consumes the struct.
  }
  fn getColor(&self) -> Color {
    // ... Note: &self just borrows as normal.
  }
}
```

### Traits

```rs
trait Codec {
  type Item;
  fn decode(bytes: Vec<u8>) -> Self::Item;
  fn encode(self) -> Vec<u8>;
}

impl Codec for u32 {
  type Item = u32;
  fn decode(bytes: Vec<u8>) -> Self::Item { ... }
  fn decode(self) -> Vec<u8> { ... }
}
```

### Enums

The following `PowerState` Enum has variants `Extenal`, `Battery` and `Dead`.

```rs 
enum PowerState {
  External,
  Battery,
  Dead
}

impl PowerState {
  fn battery_voltage(&self) -> f64 {
    match *self {
      PowerState::External => Voltage(5.0),
      PowerState::Battery => Voltage(3.6),
      PowerState::Dead => 0.0
    }
  }
}
```

Enums can contain data:

```rs 
enum PowerState {
  External(ExternalPower),
  Battery(BatteryState),
  Dead
}

impl PowerState {
  fn battery_voltage(&self) -> f64 {
    match *self {
      PowerState::External(v) => v.battery_voltage(),
      PowerState::Battery(v) => v.voltage(),
      PowerState::Dead => 0.0
    }
  }
}
```

### Useful Derive Macros

Derive is a useful attribute macro which applies addtional properties to the declaration of structs/enums.

The format of the derive macro is `#[derive(TraitName)]`. 

```rs
#[derive(Debug)]      // Debug printing.
#[derive(PartialEq)]  // Comparison of fields/values.
#[derive(PartialOrd)] // Lt/Gt of fields/values.
```

### Closures

```rs
let line = |x| x * 2 - 3;
let y0 = line(4);          // y0 = 5
let linear = |x, m, c| x * m + c;
let y1 = linear(4, 2, -3); // y1 = 5
let y2 = line(4.5, 2, -3); // error, expects integer  
```

Closures can borrow variables in the current scope:

```rs
let m = 2;
let c = -3;
let line = |x| x * m + c;
let y = line(4);   // y = 5
```

Closures compile down to structs with the necessary borrows. They have lifetime to enable local borrows.

Move closures won't borrow, but will instead take the local values. These look like `let c = move |args| { ... }`

