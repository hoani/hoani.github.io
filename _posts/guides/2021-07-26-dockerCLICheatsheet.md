---
title: "Docker CLI Cheatsheet"
excerpt: "Common docker cli commands"
toc: true
categories:
  - guide
  - software-guide
---

### Build an image

```sh
docker build -t myImage </path/to/dockerfile>
```

### Finding a container's name

```sh
docker ps
```

### Shutting down a container

```sh
docker stop <name>
docker rm <name>
```

Or:

```sh
docker rm -f <name>
```

### Volumes

Creating a volume
```sh
docker volume create myVolume
docker volume inspect myVolume
```

Running an image connected to the volume
```sh
docker run -dp 3000:3000 -v myVolume:</path/on/container> <image-name>
```

### Bind mounts

Running an image with a mount on the host machine
```sh
docker run -dp 3000:3000 -v </path/on/host>:</path/on/container> <image-name>
```

### Look at logs

```sh
docker logs <name>
docker logs -f <name> # Follow
docker logs -t <name> # Timestamp
```

### Network

```sh
docker network create myNetwork
```

Running the image on `myNetwork` with `myNetworkID` which can be used to connect to the image.
```sh
docker run -d --network myNetwork --networkAlias myNetworkID <image-name>
```


