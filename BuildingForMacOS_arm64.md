# Build on macOS* Systems for Apple Silicon

This guide shows how to build OpenVINO Runtime for later inference on Apple Silicon & Intel MYRIAD devices on OSX.
There are two options how to achieve this:
- (Native) Compile OpenVINO for arm64 architecture with extra module [OpenVINO ARM plugin] location in [OpenVINO Contrib]. Note, [Build Steps](#build-steps) will cover this as a default scenario.
- (Rosetta) Compile Intel CPU plugin `x86_64` architecture and run under [Rosetta].

The software was validated on:
- macOS\* 11.x, 12.x, arm64

## Table of content

  - [Software Requirements](#software-requirements)
  - [Build with OpenVINO ARM Plugin](#build-steps)
  - [Building x86_64 with OpenVINO Intel CPU plugin](#building-x86_64-binaries)

### Software Requirements

- [brew] package manager to install additional dependencies. Use [install brew](https://brew.sh) guide to achieve this.

- The step to install python and python libraries is different depending on host architecture:
  - **arm64** Python\* 3.6 or higher for the OpenVINO Runtime Python API, Development tools (Model Optimizer, POT and others):
  ```sh
  % # let's have a look what python versions are available in brew
  % brew search python
  % # select preferred version of python based on available ones, e.g. 3.10
  % brew install python@3.10
  ```
  - **x86_64** Select universal2 installer from [Python releases] download page and install `python-3.X.Y-macos11.pkg` image. This allows to have universal python libraries and build x86_64 OpenVINO Python API and Development tools.

- [CMake]\* 3.13 or higher:
  ```sh
  % brew install cmake
  ```
- Clang\* compiler, git and other command line tools from Xcode\* 10.1 or higher:
  ```sh
  % xcode-select --install
  % brew install git-lfs
  ```
- (arm64 only) `scons` to build ARM compute library:
  ```sh
  % python3 -m pip install scons
  ```
- (arm64 only) libusb library for MYRIAD device and `pkg-config` which is used to find `libusb` files:
  ```sh
  % brew install pkg-config libusb
  ```
- (arm64 only) TBB library for threading:
  ```sh
  % brew install tbb
  ```
- Additional `pip` dependencies to build OpenVINO Runtime Python API, Development tools (Model Optimizer, POT and others):
  ```sh
  % # update pip and setuptools to newer versions
  % python3 -m pip install -U pip setuptools
  % python3 -m pip install cython
  ```
  In order to build OpenVINO Python API and Development tools as wheel packages, additionally install requirements (after OpenVINO repo clone):
  ```sh
  % python3 -m pip install -r <openvino source tree>/src/bindings/python/wheel/requirements-dev.txt
  ```

### Build Steps

1. (Get sources) Clone submodules:
```sh
% git clone https://github.com/openvinotoolkit/openvino.git
% git clone https://github.com/openvinotoolkit/openvino_contrib.git
% cd openvino_contrib
% git submodule update --init
% cd ../openvino
% git submodule update --init
```
2. Create a build folder:
```sh
%  mkdir build && cd build
```
3. (CMake configure) OpenVINO project uses a CMake-based build system. In the created `build` directory, run `cmake` to fetch project dependencies and create build rules:
```sh
% cmake -DCMAKE_BUILD_TYPE=Release -DIE_EXTRA_MODULES=../openvino_contrib/modules/arm_plugin ..
```
> **Note:** By default OpenVINO CMake scripts try to introspect the system and enable all possible functionality based on that. You can look at the CMake output and see warnings, which show that some functionality is turned off and the corresponding reason, guiding what to do to install additionally to enable unavailable functionality. Additionally, you can change CMake options to enable / disable some functionality, add / remove compilation flags, provide custom version of dependencies like TBB, PugiXML, OpenCV, Protobuf. Please, read [CMake options for custom compilation](CMakeOptionsForCustomCompilation) for this information.
4. (CMake build) Build OpenVINO project:
```sh
% cmake --build . --config Release --parallel $(sysctl -n hw.ncpu)
```
All built binaries are located in `<openvino_source_dir>/bin/<arm64 | intel64>/Release/` and wheel packages are located in `<openvino_build_dir>/wheels`. 

5. (Optional install) Once you have built OpenVINO, you can install artifacts to a preferred location:
```sh
% cmake -DCMAKE_INSTALL_PREFIX=<installation location> -P cmake_install.cmake
```

### Building x86_64 binaries

Since OSX version 11.x and Xcode version 12.2, the Apple development tools allows to compile arm64 code on x86 hosts and vice-versa. Based on this, OpenVINO can be compiled as x86_64 binary, then run on Apple Silicon hosts using [Rosetta]. For this, first of all Rosetta must be installed:

```sh
% softwareupdate --install-rosetta
```

Then try to compile OpenVINO using the steps above, but adding `-DCMAKE_OSX_ARCHITECTURES=x86_64` on cmake configure stage. But, **don't enable any system library usage explicitly** via CMake options, because they have `arm64` architecture, e.g.:
```sh
% file /opt/homebrew/Cellar/tbb/2021.5.0_2/lib/libtbb.12.5.dylib
/opt/homebrew/Cellar/tbb/2021.5.0_2/lib/libtbb.12.5.dylib: Mach-O 64-bit dynamically linked shared library arm64
```

The same for other external dependencies like `libusb`. If you want to enable extra functionality like enable MYRIAD plugin build, you need to provide either x86_64 or universal2 `libusb` library. All other steps are the same as for usual compilation: build, install.

> **Note:** since you are building with `universal2` python libraries, wheel package is created with name `openvino-2022.3.0-000-cp39-cp39-macosx_12_0_universal2.whl` and have proper `universal2` tags, so can *potentially* be used on both Apple Silicon and Intel CPU.

[CMake]:https://cmake.org/download/
[brew]:https://brew.sh
[Rosetta]:https://support.apple.com/en-us/HT211861
[OpenVINO Contrib]: https://github.com/openvinotoolkit/openvino_contrib/tree/master
[OpenVINO ARM plugin]: https://github.com/openvinotoolkit/openvino_contrib/tree/master/modules/arm_plugin
[Python releases]: https://www.python.org/downloads/macos/
[oneTBB]: https://github.com/oneapi-src/oneTBB