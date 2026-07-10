---
title: "Docker Multiarch"
excerpt: "Using dockerx to build and run containers for different architectures"
classes: wide
permalink: /guides/software/docker-multiarch
categories:
  - guide
  - software
  - docker
---

The purpose of this article is to build and run images for different architectures. 

The examples were run on an arm64 M1 macbook with docker desktop installed.

# Installing emulators

Attempting to run an amd64 container on an arm64 machine without an emulator results in:
```sh
$ docker run --platform linux/amd64 --rm amd64/alpine uname -mo
standard_init_linux.go:228: exec user process caused: exec format error
```

We will install and run our emulators using `tonistiigi/binfmt`.

```sh
docker run --privileged --rm tonistiigi/binfmt --install all   # everything
docker run --privileged --rm tonistiigi/binfmt --install amd64 # specific
```

And now, if I run the amd64 container, it works:
```sh
$ docker run --platform linux/amd64 --rm amd64/alpine uname -mo
x86_64 Linux
```

# Building

We will start with a trivial `DockerFile`:

```dockerfile
FROM alpine:3.17.1
CMD uname -mo
```

When we use `buildx` we need to also specify an [output type](https://docs.docker.com/engine/reference/commandline/buildx_build/#output). In this case, we are targetting the development machine's docker registry.  

```sh
docker buildx build --platform linux/amd64 -t uname:amd64 . --output type=docker
```

And we can run this new container:
```sh
$ docker run --platform=linux/amd64 uname:amd64
x86_64 Linux
```

It is also important to note that this only works because the `alpine:3.17.1` image supports our target architecture. If we build for an unsupported platform, we get an error:

```sh
$ docker buildx build --platform linux/riscv64 -t uname:riscv64 . --output type=docker
...
error: failed to solve: alpine:3.17.1: no match ...
```

