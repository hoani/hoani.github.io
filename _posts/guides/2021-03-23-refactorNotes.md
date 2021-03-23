---
title: "Refactoring Notes"
excerpt: "Reference from reading Martin Fowler's Refactoring book"
toc: true
categories:
  - guide
  - software-guide
---

The following notes are from [Martin Fowler's Refactoring](https://martinfowler.com/books/refactoring.html)

## Smells

Smells are indicators of when to refactor. The following are just notes I took when reading through Refactoring.

| Smell | Description |
| ----- | ----------- |
| Mysterious Name | When it isn't obvious what something does from its name. Somtimes a sign that the design itself has issues. |
| Duplicated Code | Sometimes hides subtle differences between copies. Hard to maintain if duplicates are meant to stay in sync. |
| Long Function | Often a result of a function doing too much. Can be hard to name descriptively and hard to understand. |
| Long Parameter List | Long parameter lists obscure the meaning of a function. |
| Global Data | Hard to trace the source of unexpected changes. |
| Mutable Data | Unexpected changes can cause hard to detect and infrequent failures. |
| Divergent Change | The opposite of Single Responsibility Principle. Modules should only have one reason to change. |
| Shotgun Surgery | When responsibilities are spread across classes. E.g. multiple classes perfroming database access. |
| Feature Envy | When a function interfaces with fields/functions of another class more than its own. A sign that the function might be in the wrong class. |
| Data Clumps | Groups of data which travel together. Opportunity for grouping into a class. |
| Primitive Obsession | Using primitives to represent something which could be better represented as a class. E.g. date strings, currency, etc. |
| Repeated Switches | When there are several of the same switch logic. Possibly a sign that it's time to use polymorphism. |
| Loops | Loop logic doesn't always communicate intent well. Pipelines are more descriptive. |
| Lazy Element | A class that won't lose meaning if replaced by a function. A function that won't lose meaning if replaced by a statement. |
| Speculative Generality | Adding behaviour which doesn't have a client yet. Bloats the code base. |
| Temporary Field | An object should be expected to use all of its fields. A temporary field is only used by some objects. |
| Message Chains | When a client recursively requests object access. E.g. `student.School().Address().PostCode()` |
| Middle Man | When a class is delegating most of its behaviour to another class. |
| Insider Trading | Classes which communicate too much. Most class behaviour is better encapsulated. |
| Large Class | A sign of either duplicated code, or a class having multiple reasons to change. |
| Alternative Classes with Different Interfaces | Two classes which could share an interface but don't. Sharing an interface simplifies client code. |
| Data Class | A data class only has setters/getters. Useful manipulations are clearly being performed elsewhere. |
| Refused Bequest | When a subclass only uses a fraction of inherited data/functionality. A sign that the inheritance model is incorrect. |


## Refactorings

### Common Refactors

| Refactor | Description |
| ----- | ----------- |
| Extract Function |  |
| Inline Function |  |
| Extract Variable | Used for naming statements, aka "introduce explaining variable" |
| Inline Variable | Remove non-descriptive temporaries |
| Change Function Declaration | aka. "rename function", "change signature" |
| Encapsulate Variable | Replace shared variable access with accessor functions |
| Rename Variable |  |
| Introduce Parameter Object | Create a parameter object to group a "data clump" (a group of data that tends to travel together) |
| Combine Functions into Class |  |
| Combine Functions into Transform | Combine data computations into a transform instead |
| Split Phase | Break a block of code down into "phases" based on responsibilities. Ideally different responsibilities are grouped making other refactors easier |

### Encapsulation

| Refactor | Description |
|  ----- | ----------- |
| Encapsulate Record | Replace structures (aka "records") with classes and accessors. Allows consistency for accessing derived data |
| Encapsulate Collection | Collections include arrays, maps, dictionaries, etc |
| Replace Primitive with Object | Most primities have additional meaning which isn't represented precisely by the primitive type. Allows applying additional behaviour to the object
| Replace Temporary with Query | Replace temporary variables holding computations with a query |
| Extract Class | Break up a class with multiple responsibilities. Look for methods and data that are strongly linked |
| Inline Class | Combines two classes when they share a single responsibility anyway. Also a good inbetween refactor if the responsibilities should be split differently |
| Hide Delegate | Creates a "delegating method" to encapsulate the delegates used by a class. e.g. a class `Person` has a delegate structure `Education`, instead of allowing external users to access `person.education.grade`, provide `Person::Grade()` instead | 
| Hide Middleman | Inverse of "Hide Delegate", useful when a class has unneccessary encapsulation of classes the user already works with |
| Substitute Algorithm | Replace the body of an algorithm with another algorithm |

### Moving Features

| Refactor | Description |
| ----- | ----------- |
| Move Function | Move a function from one class to another |
| Move Field | Move a field from one class to another |
| Move Statements into Function | Move common external statements into a function |
| Move Statemenets into Caller | The inverse of "Move Statements into Function" |
| Replace Inline Code with Function Call | Replace a some statements with a pre-existing function call that does the same thing |
| Slide Statements | Move related statements together |
| Split Loop | Breaks up loops with multiple responsibilities. Makes loops easier to understand and use |
| Replace Loop with Pipeline | Improves the expressiveness of the data transform |
| Remove Dead Code | |

### Organisizing Data

| Refactor | Description |
| ----- | ----------- |
| Split Variable | When a variable is used for more than one responsibility, split it so that it handles its individual responsibilities |
| Rename Field | Rename fields of a struct/class for comprehension |
| Replace Derived Varable with Query | Query computations are safer and more repeatable than derived variables |
| Change Reference to Value | Make fields values instead of references to take control of when they change |
| Change Value to Reference | Inverse of "Change Reference to Value" when we want value updates to affect all consumers |

### Conditional Logic

| Refactor | Description |
| ----- | ----------- |
| Decompose Conditional | Apply extract function to conditional statements/conditional bodies to make them read better |
| Consolidate Conditional Expression | Consolidate a series of related checks into a function which describes the related meaning of the checks |
| Replace Nested Conditional with Guard Clauses | Use guard clauses to keep unexpected behaviour documented outside of the expected behaviour |
| Replace Conditional with Polymorphism | Add structure and extendability to complex contitional logic by putting the conditional behaviour in subclasses instead |
| Introduce Special Case | Uses a special case subclass handle special casses of a class. `Null/"unknown"` are common targets for this |
| Introduce Assertion | Communicate assumptions through assertions |

### Refactoring APIs

| Refactor | Description |
| ----- | ----------- |
| Separate Query from Modifier | Functions shouldn't modify observable state *and* return data. Separates the modifying responsibilities from the querying responsibilities |
| Parameterize Function | Combines several functions with similar logic using a parameter |
| Remote Flag Argument | Flag arguments determine how a function executes. Instead of flag arguments, break functions into a set of better named functions |
| Preserve Whole Object | When a function uses derived data, consider just passing the original object instead |
| Replace Parameter with Query | Useful when the function could determine the parameter on its own without having the parameter passed in |
| Replace Query with Parameter | Inverse of "Replace Parameter with Query" when we want to manipulate the dependencies |
| Remove Setting Method | Some fields we don't want to change once the object is instantiated |
| Replace Constructor with Factory Functions | Factory functions are more flexible and can return subclasses when appropriate |
| Replace Function with Command | A "Command" is an object with an `execute()` function. Enables slightly more flexibility and inheritance. Only useful in special cases |
| Replace Command with Function | Inverse of "Replace Function with Command" |

### Dealing with Inheritance

| Refactor | Description |
| ----- | ----------- |
| Pull Up Method | Pull up a method from the subclasses into the superclass. Useful when subclasses share a method which makes more sense in the superclass |
| Pull Up Field | Pull up a field from the subclasses into the superclass |
| Pull Up Constructor Body | Move useful statements shared in subclasses into the superclass constructor |
| Push Down Method | Push down a method from the superclass into the subclass(es). Useful when superclasses have methods that are more specific to only some/one subclass |
| Push Down Field | Push down a field from the superclass into the subclass(es) |
| Replace Type Code with Subclasses | Use subclasses instead of a type field. Allows breaking up specific fields and applying "Replace Conditional with Polymorphism". If the class already has subclasses, can make a Type class with its subclasses and behaviour instead |
| Remove Subclass | Absorb a subclass doing too little into its superclass |
| Extract Superclass | Absorb shared behaviour between subclasses into the superclass |
| Collapse Heirachy | Merge a subclass/superclass pair together. Useful when differences are trivial |
| Replace Subclass with Delegate | Use a delegate (another object) which provides the functionality of the removed subclass. Enables object composition over inhertance |
| Replace Superclass with Delegate | Inheritance can couple functions which aren't important to the subclass; when this happens, use a delegate instead |
