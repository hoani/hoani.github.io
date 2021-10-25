---
title: "Shell commands"
excerpt: "A bunch of shell commands I find useful"
toc: true
categories:
  - guide
  - software-guide
---

See [Bash Scripting Cheatsheet](https://devhints.io/bash) for a whole collection of bash tricks.

## Logging

Pipe data to a log file and stdout:

```sh
somecommand 2>&1 | tee logfile.txt
```

Reading the log file with shell colors:

```sh
less -R logfile.txt
```

Split a large logfile.txt into smaller logs, 1000 lines each:

```sh
split -l 1000 logfile.txt split
```

## Finding

To find a file with a particular name, use:
```sh
find <searchroot> -name filename
```

Using `$(pwd)` lets us search the current directory.

```sh
find $(pwd) -name myfile.txt
```

## Grep filtering

Use `-A <N>` and `-B <N>` to show `<N>` lines of information after or before the searched text.

This is useful when automating tasks or executing tests. For example, in golang we can invoke tests to run many times using the `count` command. In golang, concurrency test bugs can be sniffed out by running a test many times. If we want to run the test `TestThis` 1000 times, but only show failures, we can use:

```sh
go test -run=TestThis -v -count=1000 | grep -B 3 FAIL
```

## Disk usage

To display overall system disk usage (note: `-h` for human readable):

```sh
df -h
```

To display disk usage of the current folder (note: `-s` for simplify):

```sh
du -hs $(pwd)
```

To display disk usage of all subdirectories:

```sh
du -h $(pwd)
```

## Multiple execution

Found on [askubuntu](https://askubuntu.com/a/539293)

```sh
A; B    # Run A and then B, regardless of success of A
A && B  # Run B if and only if A succeeded
A || B  # Run B if and only if A failed
A &     # Run A in background.
```

## SSH

To avoid entering a passphrase each time you use ssh, add the following to `~/.bashrc`:

```sh
alias sshlogin='eval $(ssh-agent); ssh-add ~/.ssh/<key_file>`
```

Use `sshlogin` to only need to enter the passphrase once.


