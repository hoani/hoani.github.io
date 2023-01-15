---
title: "Python Decorators"
excerpt: "Using decorators in python"
permalink: /guides/software/python/decorator
classes: wide
categories:
  - guide
  - python
  - python-guide
---

Decorators let us extend a function's behaviour.

```python
def big_red(f):
    def wrapper(*args, **kwargs):
        print('\x1b[1;31m', end='')
        f(*args, **kwargs)
        print('\x1b[0m', end='')
    return wrapper

@big_red
def hello_number(value):
    print(f"Hello {value}!")

hello_number(5)
```

`big_red` makes a function's output turn bold and red.

The directive `@big_red` is the equivalent to:

```python
def hello_number(value):
    print(f"Hello {value}!")

hello_number = big_red(hello_number)
```

## Decorating Classes

The next example does the following:
* decorates a class rather than a function
* takes an argument

```python
def Family(surname):
    def decorator_family(cls):
        class Wrapper(cls):
            def __init__(self, name):
                super().__init__(name)
                self.name = self.name + " " + surname

        return Wrapper
    return decorator_family


@Family(surname="Jones")
class Person:
    def __init__(self, name):
        self.name = name

    def greet(self):
        print(f"Hello, I am {self.name}")


Person("Steve").greet()
```

The output is:

```sh
Hello, I am Steve Jones
```

## Classes that Decorate

Sometimes you want to write a class that decorates a function (or another class). This is useful if you want to make your decoration stateful.

```python
def Limit(max_calls=2):

    class Limit:

        def __init__(self, function):
            self.function = function
            self.calls = max_calls

        def __call__(self, *params):
            if self.calls <= 0:
                raise RuntimeError("Limit reached!")
            else:
                self.calls = self.calls - 1
                return self.function(*params)
    return Limit


@Limit(max_calls=2)
def guess_number(guess):
    if guess == 3:
        print("You guessed correctly")
    else:
        print("Bad guess!")


guess_number(1)
guess_number(2)
guess_number(3)
```

The output should read:

```sh
Bad guess!
Bad guess!
Traceback (most recent call last):
  File "/Users/hoani/sandbox/python/decorators.py", line 29, in <module>
    guess_number(3)
  File "/Users/hoani/sandbox/python/decorators.py", line 12, in __call__
    raise RuntimeError("Limit reached!")
RuntimeError: Limit reached!
```

But if we change the decorator to `@Limit(max_calls=3)`; then we get:

```sh
Bad guess!
Bad guess!
You guessed correctly
```