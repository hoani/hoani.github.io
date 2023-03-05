---
title: "UML Class Diagrams"
excerpt: "Cheatsheet for UML Class Diagrams"
permalink: /guides/software/uml
toc: true
categories:
  - guide
  - software
---

## The Class Element

### Structure

![](/assets/images/posts/guides/umlClass/000-structure.png){: .rounded-corners}

* Attributes = variables, fields, parameters
* Operations = functions, methods

### Visibility Notation

![](/assets/images/posts/guides/umlClass/001-visibility.png){: .rounded-corners}

* `+` = public
* `-` = private
* `#` = protected

## Relationships

### Inheritance

![](/assets/images/posts/guides/umlClass/100-inheritance.png){: .rounded-corners}

* `<<Name>>` = an abstract class

### Association

![](/assets/images/posts/guides/umlClass/101-association.png){: .rounded-corners}

* How a class uses another

### Aggregation

![](/assets/images/posts/guides/umlClass/102-aggregation.png){: .rounded-corners}

* Potential membership of one class to another

### Composition

![](/assets/images/posts/guides/umlClass/103-composition.png){: .rounded-corners}

* Indicates that a class only exists as a member of another class

### Multiplicity

![](/assets/images/posts/guides/umlClass/104-multiplicity.png){: .rounded-corners}

* `1` = must be one
* `3` = must be 3
* `*` = any number
* `2..*` = at least 2
* `1..5` = one to 5



