---
title: "Python Argparse"
excerpt: "CLI argparse cheatsheet"
permalink: /guides/software/python/argparse
toc: true
categories:
  - guide
  - python
---

See the [official docs](https://docs.python.org/3/library/argparse.html#name-or-flags) for the full API.

## Basic Parsing

```python
import argparse

parser = argparse.ArgumentParser(description='Simple CLI')
parser.add_argument("name")

args = parser.parse_args() # Returns a Namespace
print(f"Hello {args.name}")
```

Produces:
```sh
$ python3 cli.py
usage: cli.py [-h] name
cli.py: error: the following arguments are required: name
$ python3 cli.py --help
usage: cli.py [-h] name

Simple CLI

positional arguments:
  name

optional arguments:
  -h, --help  show this help message and exit
$ python3 cli.py Hoani
Hello Hoani
```

## Optional Arguments

Optional arguments are prefixed by `--`:

```python
parser = argparse.ArgumentParser(description='Simple CLI')
parser.add_argument("name")
parser.add_argument("--age")

args = parser.parse_args()
print(f"Hello {args.name}", f"you are {args.age}" \
      if args.age else "")
```

Produces:
```sh
$ python3 cli.py Hoani --age 31
Hello Hoani you are 31
$ python3 cli.py Hoani
Hello Hoani
```

## Argument options

* `default` - if the arg isn't specified, this value is used
* `type` - specify the type
* `help` - help string
* `nargs` - the number of arguments
    * `*` => zero or more
    * `+` => one or more
    * `1..N` => explicit number of args
        * all of the above values turns an argument into a List
* `dest` - change the name in the returned Namespace
* `metavar` - change the name used in help messages
* `choices` - limit what values can be used
    * e.g. `choices=['tcp','bluetooth','uart']`

## Subparsers

Sometimes we want to write a cli which provides several services. For example, a simple remote get/set API.

```sh
python3 cli.py [ADDR] get [NAME] 
python3 cli.py [ADDR] set [NAME] [VALUE] 
```

```python
import argparse

parser = argparse.ArgumentParser(description='Control')
parser.add_argument("addr")
subparsers = parser.add_subparsers(dest="command")
parser_get = subparsers.add_parser('get')
parser_get.add_argument("name")
parser_set = subparsers.add_parser('set')
parser_set.add_argument("name")
parser_set.add_argument("value")

args = parser.parse_args()
print(args)
```

Produces:
```sh
$ python3 cli.py localhost:1234         
Namespace(addr='localhost:1234', command=None)
$ python3 cli.py localhost:1234 get led
Namespace(addr='localhost:1234', command='get', name='led')
$ python3 cli.py localhost:1234 set led ON
Namespace(addr='localhost:1234', command='set', name='led', 
value='ON')
```

## Unit testing a parser

When testing, we can pass a string of arguments to `parse_args`:

```python
parser.parse_args(["hoani", "--age", "32"])
```
