---
title: "Rust Generics"
excerpt: "Using Generics in Rust"
classes: wide
permalink: /guides/software/rust/generics
categories:
  - guide
  - rust
  - software
---

Generics typically start with `<T>` by convention and are use to generalize types.

If we wanted to use the same 3-axis accelerometer struct for both raw data and floating point data we could use:
```rust
struct Accelerometer<T>{
	x: T,
	y: T,
	z: T,
}
//...
let raw_data = Accelerometer { x: 10_i16, y: -23_i16, z: 9812_i16 };
let data = Accelerometer { x: 0.01, y: -0.023, z: 9.812 };
```

Generics can be used with enums:
```rust
enum ImuSensor<T, U, V> {
	Accelerometer<T>,
	Gyroscope<U>,
	Magnetometer<V>,
};
```

Implementing structs or enums with Generics:
```rust
impl<T> Accelerometer<T>
where T: std::ops::Add { //..
```

Functions can take generic types too:
```rust
fn calculate_angle<T>(x: T, y: T) -> f32 { //..
```

Generic types should have constraints applied to ensure some interfaces are avaliable to them:
```rust
fn calculate_angle<T: std::convert::Into<f32>>(x: T, y: T) -> f32 { //..
// Alternatively:
fn calculate_angle<T>(x: T, y: T) -> f32
where
	T: std::convert::Into<f32>
{ //..
```