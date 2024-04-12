---
title: "An OpenGL Skeleton Code with CMake"
description: How to use my template for opengl project
slug: opengl-cmake-skeleton
date: 2023-08-14T13:44:37-07:00
image: 
categories:
  - Computer Graphics
  - OpenGL
  - CMake
tags:
  - Computer Graphics
  - OpenGL
  - CMake
---

### TL;DR Summary

Check out [sushruta/starter-code-opengl](https://github.com/sushruta/starter-code-opengl) repository and use it as a template for your next OpenGL project with code laid out neatly and out of the box support for GLFW, GLM and GLAD.

### Background

Started writing code in OpenGL code and I quickly realized that there are no good CMake resources anywhere. There are ones with Makefile or ones that describe the steps but none of them present a CMake file that works with GLAD, GLFW3 and GLM.

In the future, I will also be writing OpenGL code that interacts with CUDA and the complexity of the build process would only increase even more. That's why I decided to invest some time in creating a CMake project that I can use and reuse for various OpenGL applications. As I am (re-)learning CUDA and attempt to write a rasterizer in CUDA and OpenGL, I will also integrate the CUDA related changes in the CMake file.

### Pre-requisities

#### Things that need to be installed

1. `cmake` - I used apt-get to install it on my ubuntu machine.
2. `curl` or `wget` to download things from the internet
3. `git` to do code version control
4. Latest Driver for your GPU. Changes happen quite frequently and it is always good to keep your GPU driver current.

#### Installation of GLFW

Please download the [source code of GLFW](https://www.glfw.org/download) and install it using cmake. Do -

```
cd <glfw-source-directory>
mkdir build; cd build
cmake ..
make
sudo make install
```

The above steps will install GLFW somewhere in `/usr/local/...`. Of special interest is the `.cmake` file that is also installed. We will use it later as you will see.

#### Installation of GLM

Get the [source code for GLM](https://github.com/g-truc/glm) and follow a similar process as above -

```
... go to the source, create a build directory ...
cmake ..
make
sudo make install
```

This will install GLM's headers in `/usr/local/...` something and the cmake file will again be available for us to use later on.

#### Get GLAD

Go to [OpenGL loader/generator webpage](https://glad.dav1d.de/) and choose from language, spec, version and profile. Then click generate to get a zip that contains some code.

I choose `C++` in language, `OpenGL` in specification, `4.5` in version and `Core` in profile.

It is convenient to add `glad` directory in the project folder itself. You could also consider adding it at global level but that will involve doing different things based on the OS. This is how the glad code is laid out for me -

```
project-root-folder
  |
  - glad
    |
    - include
      |
      - glad
        |
        -glad.h
      - KHR
    - src
      |
      - glad.c
...
```

#### Get stb headers for various image and text utility functions

Just create a folder called `stb` and add two folders `src` and `include`. Look at [this directory on github](https://github.com/sushruta/starter-code-opengl/tree/main/stb) to understand this more.

### The Complete Code Layout

In the [root project folder](https://github.com/sushruta/starter-code-opengl), we have -

1. `src` directory holding the `.cpp` files.
2. `include` directory holding the `.h` and `.hpp` files
3. `glad` directory containing glad related files
4. `resources` directory containing textures and shaders.
5. `stb` containing image related methods.

All of the above directories except `resources` and `include` contain their own `CMakeLists.txt` file.

### The Role of Different CMakeLists.txt Files

I will point out the most interesting lines in each CMakeLists file.

In the root CMakeLists file, there are lines like these -

```
set(glfw3_DIR /usr/local/lib/cmake/glfw3)
```

These rely on the `.cmake` files installed earlier and they get all the include and linking information from there.

```
... REQUIRED)
```

allows us to fail and stop the process if the libraries are not found.

```
set(CMAKE_RUNTIME_OUTPUT_DIRECTORY ${CMAKE_BINARY_DIR})
```

allows us to set the location where the executable sits. I want the executable to sit in the root build directory and hence made that specification here.

The CMakeLists file in stb and glad give the directions of how to generate a static lib from the files and use it while creating the top level executable.

```
target_link_libraries(app PUBLIC glad stb glfw3 GL X11)
```

in the CMakeLists.txt file of src tells what libraries are needed to complete the linking process.

### Link to the Github Repo

The code is available for use [here in github](https://github.com/sushruta/starter-code-opengl). Please take a look at it and use it if you think it works for you. It provides out of box integration with -

Feel free to create an issue there if you think some functionality is missing.
