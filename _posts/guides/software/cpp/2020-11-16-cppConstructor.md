---
title: "C++ Constructor Patterns"
excerpt: "Simple C++ constructor patterns"
permalink: /guides/software/cpp/constructor
toc: true
categories:
  - guide
  - cpp
  - software
---

### Default

Setting a `default` constructor allows instantiation of an object using the default values of the class member variables.

```cpp
class Wheel {
public:
  Wheel(void) = default;

  double radius = 1.0;
  double mass = 0.0;
};

int main() {
  Wheel myWheel {};
  std::cout << myWheel.radius << std::endl; // 1.0 
  std::cout << myWheel.mass << std::endl;   // 0.0
}
```
[Run on cpp.sh](http://cpp.sh/4lcf2)

*Note: the above code also runs when no constructor is defined; this isn't always the case and depends on data members, so it is best to use `default` when that is the desired behaviour*

### Private/Delete

Sometimes, we want a class to not have a constructor. Maybe we want it as a base class or maybe we only want it for it's static members.

```cpp
class WheelBase {
public:
  WheelBase(void) = delete;

  double radius = 1.0;
  double mass = 0.0;
};

class PanelBase {
public:
  double area = 1.0;
private:
    PanelBase(void);
};

int main() {
  WheelBase myWheel; // Compile error
  PanelBase myPanel; // Compile error
}
```
[Run on cpp.sh](http://cpp.sh/37cbhv)

*The private constructor is also an important part of implementing the [singleton pattern](https://stackoverflow.com/questions/1008019/c-singleton-design-pattern).*

### Explicit

The `explicit` specifier prevents instantiation of a class using a conversion, and is only really used for constructors with a single parameter.

```cpp
class ConvertingPanel {
public:
  ConvertingPanel(double areaIn) : area(areaIn) {}
  double area = 1.0;
};

class ExplicitPanel {
public:
  explicit ExplicitPanel(double areaIn) : area(areaIn) {}
  double area = 1.0;
};

int main() {
  ImplicitPanel panel2 = 0.0; // OK
  ExplicitPanel panel1 = 1.0; // Compile error
}
```
[Run on cpp.sh](http://cpp.sh/37sid)

*[Converting constructors](https://en.cppreference.com/w/cpp/language/converting_constructor) are generally regarded as bad practice since their behaviour is not always intuitive.*


