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

### Basic

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

### Encapsulation Refactors

