---
title: "Dart cheatsheet"
excerpt: "Cheatsheet for dart"
toc: true
categories:
  - guide
  - dart
  - dart-guide
---

## Loop Syntax

```dart
for (int i=0; i<10; i++){
  // do something
}
```

```dart
int i = 0;
while (i++ < 10){
  // do something
}
```

## String Interpolation

```dart
var name = "Hoani";
var age = 33;
print("$name is $age."); // Hoani is 33.
```

With a struct, we need to use `${}`:
```dart
var user = Person(name: "Hoani", age: 33);
print("${user.name} is ${user.age}.");
```

## Variable Declarations

```dart
int i = 0;
String product = "shampoo";
double price = 3.99;
```

The type can be inferred using `var`:
```dart
var i = 0;
var product = "shampoo";
var price = 3.99;
```

`final` declares immutable values:
```dart
final mass = 5.972e24;
final planet = "Earth";
```

`dynamic` is used for variables whose type is not known:
```dart
dynamic code = 1234;
code = "abcd";
```

## Functions

```dart
void doSomething() {
  // do something.
}

String formatProduct(Product product) {
  return "${product.name} costs ${product.price}";
}
```

Optional arguments:
```dart
// formatPlanet("Earth", 5.972e24); 
//         -> "Planet Earth, mass: 5.972e+24kg"
// formatPlanet("Mars");            
//         -> "Planet Mars"
String formatPlanet(String name, [double mass=0.0]) {
  return "Planet $name" + 
    ((mass <= 0.0) ? "" : ", mass: $masskg");
}
```

Named arguments:
```dart
// formatPlanet("Earth", mass: 5.972e24); 
//         -> "Planet Earth, mass: 5.972e+24kg"
// formatPlanet("Mars");            
//         -> "Planet Mars"
String formatPlanet(String name, {double mass=0.0}) {
  return "Planet $name" + 
    ((mass <= 0.0) ? "" : ", mass: $masskg");
}
```

Anonymous functions:
```dart
(String name) => "Planet $name";
```

## Lists

```dart
var fibonacci = [0, 1, 1, 2, 3, 5, 8];
fibonnaci.addAll([13, 21]);
fionnaci.add(34);
```

Explicit type declaration:
```dart
List<int> fibonacci = [0, 1, 1, 2, 3, 5, 8];
```

Iterate through a list:
```dart
for (int value in fibonacci) {
  print(value);
}
```

Using `fold` to sum a list:
```dart
int sum = fibonacci.fold(0, (int s, int v) => s + v);
```

## Map syntax

```dart
var users = {
  'Steve': Person('Steve', age: 37),
  'Dave': Person('Dave', age: 27),  
};

var user = users["Steve"]; // Person('Steve', age: 37);
var user = users["Kelly"]; // null;
```

Explicit type declaration:
```dart
Map<String, Person> users = {
  'Steve': Person('Steve', age: 37),
  'Dave': Person('Dave', age: 27),  
};
```

Iterating through a map:
```dart
for (String key in users.keys) {
  print("$key");
}
```

## Classes

Simple struct-like class:

```dart
class Person{
  String name = "noone";
  int age = 0;
}

var user = Person();
user.name = "Hoani";
user.age = 33;
```

Class with constructor and immutable fields:

```dart
class Person{
  Person(this.name, this.age);

  final String name;
  final int age;
}

var user = Person("Hoani", 33);
```

### Member Functions

```dart
class Person{
  Person(this.name, this.age);

  final String name;
  final int age;
  
  String toString() => "$name is $age.";
}

var user = Person("Hoani", 33);
print(user); // Calls toString, prints "Hoani is 33."
```

### Private fields and functions

```dart
class Person{
  Person(this._name, this._age);

  final String _name; // Private
  final int _age; // Private
  
  String toString() => "$_name is ${_printAge()}.";
  String _printAge() => "$_age years old"; // Private
}

var user = Person("Hoani", 33);
print(user); // Prints "Hoani is 33 years old."
```

### Inheritance

```dart
class CatOwner extends Person{
  CatOwner(String name, int age, this._catName) : 
    super(name, age);

  final String _catName;
  
  String toString() => super.toString() + 
              " His cat is $_catName.";
}

var user = CatOwner("Hoani", 33, "Indie");
print(user); // "Hoani is 33 years old. His cat is Indie."
```

### Abstract Classes

```dart
abstract class Vector {
  double distance();
  int dimensions();
}

class Vector3 implements Vector {
  Vector3(this.x, this.y, this.z);

  double x = 0.0, y = 0.0, z = 0.0;

  double distance() => sqrt(x * x + y * y + z * z);
  int dimensions() => 3;
}
```

### Computed Properties

```dart
abstract class Vector {
  double get distance;
  int get dimensions;
}

class Vector3 implements Vector {
  Vector3(this.x, this.y, this.z);

  double x = 0.0, y = 0.0, z = 0.0;

  double get distance => sqrt(x * x + y * y + z * z);
  int get dimensions => 3;
}
```

### Mixins

Mixins extend the functionality of classes.

```dart
mixin Discount {
  double applyDiscount(double price, double percent) {
    return price * (1.0 - (percent/100.0));
  }
}

class StoreItem with Discount {
  // ...
  double get price => applyDiscount(_basePrice, _discount);
}
```

## Enums

```dart
enum Command {
    Sit,
    RollOver,
};

void handleCommand(Command cmd) {
  switch (cmd) {
    case Command.Sit:
      // do sit
      break;
    case Command.Rollover:
      // do rollover
      break;
    default:
      print("Unknown Command");
      break;
  } 
}
```

## Streams

```dart
import 'dart:async';

void main() {
  final controller = StreamController();

  controller.sink.add("Hello");
  controller.sink.add(null);
  controller.sink.add(1234);
  controller.sink.addError(StateError("something is wrong"));
  controller.close();

  controller.stream.listen(
    (v)            => print(v),
    onError: (err) => print("Error: \t" + err.toString()),
    onDone: ()     => print("stream closed"),
  );
}
```

Output:

```
Hello
null
1234
Error: 	Bad state: something went wrong
stream closed
```
