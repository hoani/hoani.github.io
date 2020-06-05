---
title: "C++ Classes"
excerpt: "Inheritance, Virtual Functions and Interfaces"
classes: wide
tags: cheatsheet cpp
---

Inheritance:
```cpp
class Accelerometer : public Vec3 { 
  //...
```

Virtual functions allow dynamic dispatch so that we can override a function even when we cast the inherited class to its base class.
* Requires additional memory
* Takes extra time to do the vtable lookup

```cpp
class Vec3 {
public:
  float x,y,z;
  virtual float Norm();
}

class Accelerometer {
public:
  float Norm() override;
}
```

Pure virtual functions allow us to create interface specifications:
```cpp
class IVec {
public:
  virtual float Norm() = 0;
}
```

