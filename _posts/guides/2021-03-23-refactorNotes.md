---
title: "Refactoring Notes"
excerpt: ""
toc: true
categories:
  - guide
  - software-guide
---

The following notes are from [Martin Fowler's Refactoring](https://martinfowler.com/books/refactoring.html)

## Refactorings

### Common Refactors

#### Extract Function

#### Inline Function

#### Extract Variable

* Used for naming statements
* aka. "introduce explaining variable"

#### Inline Variable

* Remove non-descriptive temporaries

#### Change Function Declaration

* aka. "rename function", "change signature"

#### Encapsulate Variable

* Replace shared variable access with accessor functions.

#### Rename Variable

#### Introduce Parameter Object

* Create a parameter object to group a "data clump" (a group of data that tends to travel together)

#### Combine Functions into Class

#### Combine Functions into Transform

* Combine data computations into a transform instead

#### Split Phase

* Break a block of code down into "phases" based on responsibilities.
* Ideally different responsibilities are grouped making other refactors easier.

### Encapsulation

#### Encapsulate Record

* Replace structures (aka "records") with classes and accessors
* Allows consistency for accessing derived data

#### Encapsulate Collection

* Collections include arrays, maps, dictionaries, etc.

#### Replace Primitive with Object

* Most primities have additional meaning which isn't represented precisely by the primitive type
* Allows applying additional behaviour to the object

#### Replace Temporary with Query

* Replace temporary variables holding computations with a query

#### Extract Class

* Break up a class with multiple responsibilieites
* Look for methods and data that are strongly linked

#### Inline Class

* Combines two classes when they share a single responsibility anyway
* Also a good inbetween refactor if the responsibilities should be split differently

#### Hide Delegate

* Creates a "delegating method" to encapsulate the delegates used by a class
* e.g. a class `Person` has a delegate structure `Education`, instead of allowing external users to access `person.education.grade`, provide `Person::Grade()` instead

#### Hide Middleman

* Inverse of "Hide Delegate", useful when a class has unneccessary encapsulation of classes the user already works with

#### Substitute Algorithm

* Replace the body of an algorithm with another algorithm

### Moving Features

#### Move Function

* Move a function from one class to another

#### Move Field

* Move a field from one class to another

#### Move Statements into Function

* Move common external statements into a function

#### Move Statemenets into Caller

* The inverse of "Move Statements into Function"

#### Replace Inline Code with Function Call

* Replace a some statements with a pre-existing function call that does the same thing

#### Slide Statements

* Move related statements together

#### Split Loop

* Breaks up loops with multiple responsibilities
* Makes loops easier to understand and use

#### Replace Loop with Pipeline

* Improves the expressiveness of the data transform

#### Remove Dead Code

### Organisizing Data

#### Split Variable

* When a variable is used for more than one responsibility, split it so that it handles its individual responsibilities

#### Rename Field

* Rename fields of a struct/class for comprehension

#### Replace Derived Varable with Query

* Query computations are safer and more repeatable than derived variables

#### Change Reference to Value

* Make fields values instead of references to take control of when they change

#### Change Value to Reference

* Inverse of "Change Reference to Value" when we want value updates to affect all consumers

### Conditional Logic

#### Decompose Conditional

* Apply extract function to conditional statements/conditional bodies to make them read better

#### Consolidate Conditional Expression

* Consolidate a series of related checks into a function which describes the related meaning of the checks

#### Replace Nested Conditional with Guard Clauses

* Use guard clauses to keep unexpected behaviour documented outside of the expected behaviour

#### Replace Conditional with Polymorphism

* Add structure and extendability to complex contitional logic by putting the conditional behaviour in subclasses instead

#### Introduce Special Case

* Uses a special case subclass handle special casses of a class
* Null/"unknown" are common targets for this

#### Introduce Assertion

* Communicate assumptions through assertions

### Refactoring APIs

#### Separate Query from Modifier








