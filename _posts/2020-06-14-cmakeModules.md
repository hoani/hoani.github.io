---
title: "CMake Modulation"
excerpt: "Working with static libraries"
classes: wide
categories:
  - guide
  - cpp
  - cpp-guide
  - cmake
  - cmake-guide
---

Most CMake projects will start to break the project up:
```
.
+-- CMakeLists.txt
+-- src
|   +-- main.cpp
|   +-- drivers
|   |   +-- CMakeLists.txt
|   |   +-- bldc.cpp
|   |   +-- bldc.hpp
|   |   +-- currentSense.cpp
|   |   +-- currentSense.hpp
+-- test
|   +-- mock
|   +-- ..etc
```

We would like to include the bldc library in our app, but define the compilation of that library in the drivers folder.

In the root `CMakeLists.txt`:
```cmake
# ...
add_executable(MyApp src/main.cpp)
add_subdirectory(src/drivers)
target_link_libraries(MyApp bldc)
```

We can now define the `bldc` library in the `src/drivers/CMakeLists.txt`:
```cmake
add_library(bldc STATIC bldc.cpp currentSense.cpp)
target_include_directories(bldc PUBLIC ${CMAKE_CURRENT_SOURCE_DIR})
```

`target_include_directories(...)` provides include directories for anything linking to the library.

The keyword `PUBLIC` states that the include directories are part of the library's interface. If the library requires an implementation only include directory, then we can specify this with `PRIVATE`. The same is true for linking libraries with `target_link_libraries`.
