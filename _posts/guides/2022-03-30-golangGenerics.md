---
title: "Go Generics"
excerpt: "Generics Overview for Golang"
toc: true
categories:
  - guide
  - golang
  - golang-guide
---

The generics feature in this guide required go 1.18 or higher.

## Generic Functions

A function I have written several times for various packages is a `Has` function:

```golang
func Has(items []string, target string) bool {
	for _, item := range items {
		if item == target {
			return true
		}
	}
	return false
}
```

Without generics, we need to rewrite this function for every type we need to search for; we may end up with functions like:

```golang
func HasStr(items []string, target string) bool 
func HasInt(items []int, target int) bool 
func HasF64(items []float64, target float64) bool 
```

With generics, we can define the function once:

```golang
func Has[T comparable](items []T, target T) bool {
	for _, item := range items {
		if item == target {
			return true
		}
	}
	return false
}
```

This can be called with any type `T` that enforces the constraint `comparable`. The `comparable constraint requires that:
* `T == T` is implemented
* `T != T` is implemented

For example:

```golang
list := []string{"a", "b", "c", "d", "e"}
fmt.Printf("%v: %v\n", "a", Has(list, "a")) // a: true
fmt.Printf("%v: %v\n", "h", Has(list, "h")) // h: false
```

### Type Inference 

In the above example, we used inferrence to determine what type `T` should use. This was easy for the compiler to deduce because our input arguments provide the type.

Determining `T` implicitly isn't always possible. For example, I might want a more convenient method to get a `nil` pointer to a specific type. Afterall, writing `var value *MyType` gets annoying. So I write:

```golang
func NilPtr[T any]() *T {
	return nil
}
```

In this case, type inference doesn't work, we need to explicitly name the type we will make `nil`:

```golang
value := NilPtr[int]()      // OK
value := NilPtr[struct{}]() // OK
value := NilPtr()           // Compile Error
```

## Generic Types

Generics can be applied to types. Here are two flavors we will probably see often:

```golang
type MyList[T any] []T

type MyStruct[T any] struct{
	a  T
	// ...
}
```

To use these generic types, we must specify the type. For example:

```golang
f32List := MyList[float32]{0.0, 1.5} // OK
f32List := MyList{0.0, 1.5}          // Compile Error
```

### Example: Generic Set
For the longest time, I have been frustrated by having to use `map[MyType]struct{}` for sets and having to rewrite my `Set` code in various packages. 

Now I can define:

```golang
type Set[T any] map[T]struct{}
```

And the set can have methods:

```golang
func NewSet[T comparable](values ...T) Set[T] {
	m := make(map[T]struct{}, len(values))
	for _, v := range values {
		m[v] = struct{}{}
	}
	return Set[T]{m: m}
}

func (s *Set[T]) Has(item T) bool {
	_, has := s.m[item]
	return has
}

// ... And more!
```

Which can be used like:

```golang
f64Set := NewSet(1.2, 3.5, 7.75)
fmt.Printf("%v: %v \n", 1.2, f64Set.Has(1.2)) // 1.2: true
fmt.Printf("%v: %v \n", 5.6, f64Set.Has(1.2)) // 5.6: false
```

One gotcha in go 1.18 (and maybe later versions? Who knows...) is that a member function cannot have its own generic types.

For example, we couldn't define:
```golang
func (s *Set[T]) Length[N: ~int|~uint]() N {
	return N(len(s.m))
}
```

This will result in the compile error `method must have no type parameters`.

## Constraints

A `constraint` is applied to a generic type argument to enforce what types can be used. 

If we wanted to write a `Has` fuction, we can take several approaches, helped by constraints.

### Constraint: `any`

`any` allows any type.

If we use the `any` constraint to write `Has` we get:
```golang
func Has[T any](items []T, v T, equal func(a,b T) bool) bool
```
* `any` is very vague, so we provide a `cmp` function to help `sort` determine order.

### Constraint: `comparable`

`comparable` restricts a type to only those which provide `==` and `!=` operators.

```golang
func Has[T comparable](items []T, v T) bool
```

### Constraint: Interface

Interfaces can be used as a constraint. If we have an interface:

```golang
type Cmp[T any] interface {
	Equal(other *T) bool
}
```

Then we can write:
```golang
func Has[T Cmp[T]](items []T, v T) bool
```

This looks a a bit funky, because the interface also takes a type argument.

A working type which can be used with `Has[T Cmp[T]]` is:

```golang
type MyType struct{ a int }

func (t *MyType) Equal(other *MyType) bool {
	return t.a == other.a
}
```

Another (less complicated) example would be using the `fmt.Stringer` interface as a constraint:

```go
func Format[T fmt.Stringer](item T) string
```

### Constraint: Type Sets

Type sets define potential candidate types.

```golang
// Can only use int - kind of pointless.
func Has[T int](items []T, v T) bool

// Can only use int or uint
func Has[T int | uint](items []T, v T) bool

// Can only use string or items with string as the
// underlying type.
func Has[T int | ~string](items []T, v T) bool
```

On underlying types:
* `type myString string` has the underlying type `string`
* `type myString struct{string}` has the underlying type `struct{string}` not `string`
	* attempting to use `struct{string}` as `~string` results in compile error: `myString does not implement ~string`

### Combining constraints

We can combine constraints in an interface:
```golang
type Currency interface {
	~uint64
	ISO4127Code() string
	Decimals() int
}
```

In this case, `Currency` specifies an interface which requires:
* The underlying type must be `uint64`
* `ISO4127Code` and `Decimals` must be implemented

The following type `NZD` meets these requirements:

```golang
type NZD uint64

func (c NZD) ISO4127Code() string { return "NZD" }
func (c NZD) Decimals() int       { return 2 }
```

Now, `NZD` can be used in a generic function like `CurrencyString`:

```go
func CurrencyString[T Currency](c T) string {
```

We can call `CurrencyString` like so:
```golang
fmt.Println(CurrencyString(NZD(500))) // 5.00 NZD
```
