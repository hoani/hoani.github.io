---
title: "Creating debian packages"
excerpt: "How to build packages for debian based systems"
toc: true
categories:
  - guide
  - software
  - software-guide
---

This guide is a brief tutorial on creating and using a debian package.

To start with, create a folder `~/debpkgs` to work out of.

Inside this folder, we can create the package we actually want. We name these packages as follows:
* `<name>_<version>_<arch>`
  * `<name>` can use `'-'` characters to split the name up
  * `<version>` will be something like `1.0`
  * `<arch>` will be the machine architecture
    * e.g `amd64`, `arm64` etc
    * use `all` if the package can be installed on any architecture

## say-hello example

Create a package with the file structure:

```
| - debpkgs/
  | - say-hello_1.0_all/
    | - DEBIAN/
    | - usr/
      | - bin/
```

* we use the `DEBIAN` directory to configure our package
* all other directories mimic where we want to install files on the target machine
  * in this case, we will install something in `/usr/bin`
  * other common locations we might use are:
    * `/opt/<vendor>/` for vendor specific files
    * `/usr/lib` for installing shared object `.so` files

Inside the `say-hello_1.0_all/usr/bin` directory, create a file `hello.sh` and place the following contents inside:

```bash
#!/bin/sh
echo "Hello everyone"
```

Make it executable for anyone: 
```sh
chmod 777 hello.sh
``` 

Inside the `say-hello_1.0_all/DEBIAN` directory, add a file called `control` with the following content:
```
Package: say-hello
Version: 1.0
Architecture: all
Essential: no
Priority: optional
Depends: 
Maintainer: Your Name
Description: Says hello in the terminal

```

The `control` file sets up the package definitions.
* `Package` specifies the actual package name
  * After installing the pacakge, we can remove it with `sudo apt remove say-hello`, where `say-hello` is provided here
* `Essential` is always `no` and `Priority` is usually `optional` unless your package is part of the operating system
* `Depends` allows us to name other packages which this package depends on

Our package structure will now look like:

```
| - debpkgs/
  | - say-hello_1.0_all/
    | - DEBIAN/
    | | - control
    | - usr/
      | - bin/
        | - hello.sh
```

If we move to the `debpkgs` directory, we can build the package:
```sh
dpkg-deb --build say-hello_1.0_all
```

Which builds the package file `say-hello_1.0_all.deb`.

## Inspecting a package

The easiest way to inspect a package is to call:

```
dpkg --info say-hello_1.0_all.deb
```

This will print out the info in the `control` file, with some additional information like archive size.

## Installing a package with dpkg

To install, call:
```
sudo dpkg --install say-hello_1.0_all.deb
```

Now you should be able to call `hello.sh` from any terminal on your computer:

```
> hello.sh
Hello everyone
```

To uninstall, use the package name `say-hello`:
```
sudo dpkg --remove say-hello
```

## Installing a package with apt

Installing with `apt` can be nice, because the `apt` tools will find your dependencies for you (if you listed any in the control file).

To install, call:
```
sudo apt install ./say-hello_1.0_all.deb
```

To uninstall, use the package name `say-hello`:
```
sudo apt remove say-hello
```
