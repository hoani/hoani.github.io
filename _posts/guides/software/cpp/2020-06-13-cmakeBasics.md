---
title: "CMake Basics"
excerpt: "Writing CMakeLists.txt for C++"
permalink: /guides/software/cpp/cmake
classes: wide
categories:
  - guide
  - cpp
  - cpp-guide
  - cmake
  - cmake-guide
---

Given a directory structure like:
```
.
+-- CMakeLists.txt
+-- src
|   +-- main.cpp
|   +-- ...etc
```

Add the following a `CMakeLists.txt`:
```cmake
cmake_minimum_required (VERSION 3.5)
project (MyProjectName)

# Allows us to to use MSVC
if(MSVC)
    add_compile_options("/W4" "$<$<CONFIG:RELEASE>:/O2>")
else()
    add_compile_options("-Wall" "-Wextra" "-Werror" "$<$<CONFIG:RELEASE>:-O3>")
endif()

set(CMAKE_CXX_STANDARD 11)

set(source_dir  "${PROJECT_SOURCE_DIR}/src/")
file(GLOB source_files "${source_dir}*.cpp")
add_executable(MyApp ${source_files})
```

## Building

When building from the command line:
```bash
$ mkdir build
$ cd build
$ cmake ../
$ cmake --build .
```

This will build `MyApp` (or `MyApp.exe` in Windows) in Debug mode by default. Depending on your distro it may place the binary either in the `build` directory or a `build/Debug` directory.

## Compile Options

We configure global options using `add_compile_options(...)`.

The `$<$<CONFIG:RELEASE>:OPTION>` can be used to add options based on whether you are building for debug or release.

By default we build in `Debug` mode. To build a `Release` binary:
```bash
$ cmake --build . --config Release
```

*Note: `add_compile_options` should only be used at the top of the root `CMakeLists.txt` for compile options which are appropriate for the entire project. For all target specific options, use `target_compile_options` or `target_compile_definitions` to limit flags to thier specific modules.*

## Setting C++ Version

```cmake
set(CMAKE_CXX_STANDARD 11)
```

## Defining output binaries

```cmake
add_executable(MyApp ${source_files})
```

When multiple binaries are added, then all binaries will be built when running:
```bash
$ cmake --build .
```

To build a specific binary (for example, `Test`), we can run:
```bash
$ cmake --build . --target Test
```