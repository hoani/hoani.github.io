---
title: "Component Principles"
excerpt: "Reference for component cohesion and coupling principles"
toc: true
permalink: /guides/software/component-principles
categories:
  - guide
  - software
---

The following notes are from [Uncle Bob's Clean Architecture](https://www.goodreads.com/book/show/18043011-clean-architecture).

In Clean Architecture, components are defined as groups of related classes which are revision controlled together.

## Component Cohesion

The following principles help determine how to break a product into components.

### REP - Reuse/Release Equivalence Principle

Consumers of a component library can only use that library with releases. Functionality that is grouped together must make sense as being releasable together.

### CCP - Common Closure Principle

Components are made up of functionality that change for the same reasons.  Similar to the Single Responsibility Principle. 

The purpose of this principle is to simplify the release process. We don't want divergent change in our components; or even worse, shotgun surgery across components.

### CRP - Common Reuse Principle

Gather functionality that is reused together in the same component. Separate functionality that is not reused together into seperate components. Similar to the Interface Segregation Principle.

The purpose of this principle is to make the component better for the users. We don't want the user's code to have to depend on functionality that they don't need.

## Component Coupling

The following princinciples help determine how to handle dependencies between components. 

Dependency Inversion is important to acheive these principles.

### ADP - Acyclic Dependencies Principle

Cycles are not allowed in the dependency graph.

### SDP - Stable Dependencies Principle

Depend in the direction of stability.

Instability of a component is defined mathematically:

$$ I = \frac{N_{fan-out}}{N_{fan-in}} $$

Where:
* $$ I \in [0, 1] $$ is instability
* $$ N_{fan-in} \ge 0, $$ is the number of classes this component's classes depends on.
* $$ N_{fan-out} \ge 0, $$ is the number of classes that depend on this component's classes.

Generally speaking, stable components are difficult to change and have reliable interfaces.

### SAP - Stable Abstractions Principle

Components should be as abstract as they are stable.

There's a formula for this too:

$$ A = \frac{N_a}{N_c} $$

Where:
* $$ A \in [0, 1] $$ is abstractness
* $$ N_a \ge 0, $$ is the number of abstract classes
* $$ N_c \ge N_a, $$ is the number of classes

It basically means use interfaces for stability.
