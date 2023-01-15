---
title: "GDB Cheatsheet"
excerpt: "Command cheatsheet for gdb."
toc: true
permalink: /guides/software/gdb
categories:
  - guide
  - software
  - software-guide
---

### Text User interface

For a more interactive gdb session, can use a text user interface:

```sh
(gdb) layout src
(gdb) layout asm
(gdb) layout split
```

Depending on which command was run, it will display the source, assembler or both.

![Altium Standards](/assets/images/posts/guides/gdb/gdb_tui.png)

To remove the text user interface (tui):

```sh
(gdb) tui disable
```

### Commands

Stepping through code is done with the `step` command. Optionally a number of steps `<N>` can be specified. Entering a blank command after a step will continue stepping the code.

To only step 1 assembly instruction at a time, use `stepi`.

```sh
(gdb) step
(gdb) step <N>
(gdb) s
(gdb) stepi
(gdb) si
```

To step over (rather than into):

```sh
(gdb) next
(gdb) n
```

To step out of the current function:

```sh
(gdb) finish
```

Use `continue` to run to the next breakpoint.

```sh
(gdb) continue
(gdb) c
```

Use `break` to set a breakpoint at entry to `<function>`.

```sh
(gdb) break <function>
(gdb) b <function>
```

Set a breakpoint at a particular file on a particular line.

```sh
(gdb) break <file>:<ln>
(gdb) break main.rs:14
```

To disable and enable breakpoints, use the breakpoint numbers:

```sh
(gdb) disable 1
(gdb) enable 1
```

A value of a local variable can be printed with `print`. Expressions can also be evaluated

```sh
(gdb) print <a>
(gdb) p <a> 
(gdb) print <a + b>
```

To show the value of all local variables:

```sh
(gdb) info locals
(gdb) i locals
```

To show function args:

```sh
(gdb) info args
```

To set the value of a local:

```sh
(gdb) set <variable> = <value>
```

Reset the session:

```sh
(gdb) monitor reset halt
```

Show the current backtrace:

```sh
(gdb) backtrace
```



Ending a session

```sh
(gdb) quit
```

### Remote debugging

```sh
(gdb) target remote <ipaddr>:<port>
```
