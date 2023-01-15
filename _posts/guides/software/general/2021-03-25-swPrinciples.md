---
title: "SOLID, ETC and DRY"
excerpt: "Reference for common software principles."
toc: true
permalink: /guides/software/principles
categories:
  - guide
  - software
  - software-guide
---

The following notes are from [Uncle Bob's Clean Architecture](https://www.goodreads.com/book/show/18043011-clean-architecture) and [Andy and Dave's The Pragmatic Programmer](https://www.goodreads.com/book/show/4099.The_Pragmatic_Programmer)

## SOLID Principles

The SOLID principles operate at the class (or class-like) level.

### SRP - Single Responsibility Principle

A class has only one reason to change. The reason to change should be driven by a stakeholder or group of stakeholders that using that class in a specific way.

### OCP - Open Closed Principle

Keep classes closed to change, but open for extension. When we want to extend a classes behaviour, we rely on interfaces or inheritance rather than modifying the class.

### LSP - Liskov Substitution Principle

A class which depends on another classes' interface shouldn't know or care where in the dependencies's inheritance heirachy it is interacting.

### ISP - Interface Segregation Principle

A class which depends on another classes' interface uses the entire interface. ISP and SRP help us draw lines where a class should be split.

### DIP - Dependency Inversion

Rely on abstract interfaces rather than concrete classes. Clean Architecture mentions this is only important for volatile modules; it's usually ok to rely on concrete classes which are unlikely to change (e.g. the standard library).

Dependency inversion is usually applied by having dependencies implement an interface; and having classes only depend on those interfaces.

## ETC - Easy to Change

ETC is my favorite principle from The Pragmatic Programmer. It is the end goal; All other principles are trying to make software easy to change.

Code that is easy to change keeps software "soft". ETC is one of a few key reasons to practice test driven development, and a big enabler for agile development.

## DRY - Don't Repeat Yourself

Don't repeat yourself applies to reduce code smells such as 'Duplicated Code', 'Shotgun Surgery', 'Data Clumps' and 'Repeated Switch Statement'.

Duplicated sources of truth are also a violation of DRY. For example, if two classes maintain a game's `Time` object independently; then we have an awkward DRY violation which require both classes to be updated when we want to update time.

