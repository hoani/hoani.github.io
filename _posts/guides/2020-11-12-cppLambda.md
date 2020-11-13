---
title: "C++ Lambda"
excerpt: "Simple C++ lambda examples"
categories:
  - guide
  - cpp
  - cpp-guide
---

A basic lambda which just adds two integer values is defined:
```cpp
[] (int a, int b) -> int { return a + b; }
```

Lambda anatomy:
- `[]` is the capture clause:
  - `[name]` passes a `const` clone of variable `name` which can be accessed in the body of the lambda
  - `[&name]` passes a mutable reference of `name` which can be accessed in the body of the lambda
  - `[x, y, z]` performs multiple captures
  - `[=]` allows us to capture anything in the current scope
  - `[&]` allows us to capture anything in the current scope by reference
- `(int a, int b)` are the lambda parameters.
  - Can use `auto` here to make the lambda *generic*; however for every new type used for the `auto` parameter, a new lambda will be generated.
- `-> int` specifies the return type
  - this can be omitted:
    - `[] (int a, int b) { return a + b; }`)
    - but it's a good idea to be explicit
- `{ return a + b; }` is the body
  - unlike python lambdas, we can put whatever we like here (python only allows an expression).

In c++ lambdas are objects (called *functors*) which do specific things like:
- store captures as data members
- override the `operator()` so the object is callable

*Even though lambdas are objects, it is preferable that they are kept stateless. Stateless lambdas are easier to understand and are simple to copy or pass without unexpected behavior.*

Lambda captures can be made mutable, by making the lambda mutable:
```cpp
[c] (int a, int b) mutable -> int { c++; return a + b + c; }
```

However, `c` becomes a data member of the lambda; therefore, mutating it won't mutate the original variable `c`. To mutate the original value of `c` use a reference:
```cpp
[&c] (int a, int b) -> int { c++; return a + b + c; }
```

Captures `[&]` and `[=]` are usually favored because there is no penalty for not explicitly defining what a lambda should capture. All of the captures are resolved at compile time, so the compiler will construct the object defintion on only the captures that the lambda needs.

*Like any other object which stores a reference,  lambda should not outlive the data it references.*



