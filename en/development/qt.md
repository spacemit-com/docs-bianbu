---
sidebar_position: 2
---

# Qt User Guide

## Introduction

**Qt** is a cross-platform C++ framework (C++ library) primarily used for developing graphical user interface (GUI) programs. It can also be used to develop command-line interface (CUI) programs without a GUI. 
**OpenGL ES** (OpenGL for Embedded Systems) is a cross-platform 2D/3D graphics API designed for rendering efficient 2D and 3D graphics on low-power devices. It is a subset of OpenGL (Open Graphics Library) tailored for embedded systems.  
This guide provides instructions for adapting Qt5 with OpenGL ES hardware acceleration on the SpacemiT SoC platform.

## Install Qt5 development environment

### Install C/C++ compilation environment

```shell
$ sudo apt-get update
$ sudo apt-get install build-essential
```

### Install Qt5 development tool

```shell
$ sudo apt-get install qtbase5-gles-dev qtchooser qt5-qmake qtbase5-dev-tools
```

Check the installed Qt version.

```shell
$ qmake -v
QMake version 3.1
Using Qt version 5.15.13 in /usr/lib/riscv64-linux-gnu
```

### Install Qt5 wayland plugin

```shell
$ sudo apt-get install qtwayland5
```

### Configure the Qt5 backend display service

```shell
$ export QT_QPA_PLATFORM=wayland
```

## Building and running Qt5 program

### Download `base-opensource-src-gles`

```shell
$ apt-get source qtbase-opensource-src-gles
$  tree qtbase-opensource-src-gles-5.15.13+dfsg -L 1
qtbase-opensource-src-gles-5.15.13+dfsg
├── bin
├── config_help.txt
├── config.tests
├── configure
├── configure.bat
├── configure.json
├── configure.pri
├── debian
├── dist
├── doc
├── examples
├── include
├── INSTALL
├── lib
├── LICENSE.FDL
├── LICENSE.GPL2
├── LICENSE.GPL3
├── LICENSE.GPL3-EXCEPT
├── LICENSE.LGPL3
├── LICENSE.LGPLv3
├── LICENSE.QT-LICENSE-AGREEMENT
├── mkspecs
├── qmake
├── qtbase.pro
├── src
├── sync.profile
├── tests
└── util
```

### Build `qtbase-opensource-src-gles`

#### Install dependencies

```shell
$ sudo apt-get build-dep qtbase-opensource-src-gles
```

#### Compile

```shell
$ dpkg-buildpackage -us -uc -nc -b -j4
```

### Build the example projects

#### Generate Makefile using qmake

```shell
$ cd qtbase-opensource-src-gles-5.15.13+dfsg/examples/widgets/animation/animatedtiles
$ qmake
```

#### Generate executable files using make

```shell
$ make
```

### Run the examples

```shell
$ ./animatedtiles
```

![qt](./static/qt.png)