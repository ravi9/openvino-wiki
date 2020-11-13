## Build for Raspbian Stretch* OS

> **NOTE**: Only the MYRIAD plugin is supported.

## Table of content

  - [Hardware Requirements](#hardware-requirements)
  - [Native Compilation](#native-compilation)
  - [Cross Compilation Using Docker\*](#cross-compilation-using-docker)
  - [Additional Build Options](#additional-build-options)
  - [(Optional) Additional Installation Steps for the Intel® Neural Compute Stick 2](#optional-additional-installation-steps-for-the-intel-neural-compute-stick-2)
    - [For Linux, Raspbian Stretch* OS](#for-linux-raspbian-stretch-os)

### Hardware Requirements
* Raspberry Pi\* 2 or 3 with Raspbian\* Stretch OS (32-bit). Check that it's CPU supports ARMv7 instruction set (`uname -m` command returns `armv7l`).

  > **NOTE**: Despite the Raspberry Pi\* CPU is ARMv8, 32-bit OS detects ARMv7 CPU instruction set. The default `gcc` compiler applies ARMv6 architecture flag for compatibility with lower versions of boards. For more information, run the `gcc -Q --help=target` command and refer to the description of the `-march=` option.

You can compile the Inference Engine for Raspberry Pi\* in one of the two ways:
* [Native Compilation](#native-compilation), which is the simplest way, but time-consuming
* [Cross Compilation Using Docker*](#cross-compilation-using-docker), which is the recommended way

### Native Compilation
Native compilation of the Inference Engine is the most straightforward solution. However, it might take at least one hour to complete on Raspberry Pi\* 3.

1. Install dependencies:

  ```bash
  sudo apt-get update
  sudo apt-get install -y git cmake libusb-1.0-0-dev
  ```

2. Go to the cloned `openvino` repository:

  ```bash
  cd openvino
  ```

3. Initialize submodules:

  ```bash
  git submodule update --init --recursive
  ```

4. Create a build folder:

  ```bash
  mkdir build && cd build
  ```

5. Build the Inference Engine:

  ```bash
  cmake -DCMAKE_BUILD_TYPE=Release \
        -DENABLE_SSE42=OFF \
        -DTHREADING=SEQ \
        -DENABLE_GNA=OFF .. && make
  ```

### Cross Compilation Using Docker*

  This compilation was tested on the following configuration:

  * Host: Ubuntu\* 18.04 (64-bit, Intel® Core™ i7-6700K CPU @ 4.00GHz × 8)
  * Target: Raspbian\* Stretch (32-bit, ARMv7, Raspberry Pi\* 3)

1. Install Docker\*:

  ```bash
  sudo apt-get install -y docker.io
  ```

2. Add a current user to `docker` group:

  ```bash
  sudo usermod -a -G docker $USER
  ```

  Log out and log in for this to take effect.

3. Create a directory named `ie_cross_armhf` and add a text file named `Dockerfile`
with the following content:

  ```docker
  FROM debian:stretch

  USER root

  RUN dpkg --add-architecture armhf && \
      apt-get update && \
      apt-get install -y --no-install-recommends \
      build-essential \
      crossbuild-essential-armhf \
      git \
      wget \
      libusb-1.0-0-dev:armhf \
      libgtk-3-dev:armhf \
      libavcodec-dev:armhf \
      libavformat-dev:armhf \
      libswscale-dev:armhf \
      libgstreamer1.0-dev:armhf \
      libgstreamer-plugins-base1.0-dev:armhf \
      libpython3-dev:armhf \
      python3-pip \
      python-minimal \
      python-argparse

  RUN wget https://www.cmake.org/files/v3.14/cmake-3.14.3.tar.gz && \
      tar xf cmake-3.14.3.tar.gz && \
      (cd cmake-3.14.3 && ./bootstrap --parallel=$(nproc --all) && make --jobs=$(nproc --all) && make install) && \
      rm -rf cmake-3.14.3 cmake-3.14.3.tar.gz
  ```

  It uses the Debian\* Stretch (Debian 9) OS for compilation because it is a base of the Raspbian\* Stretch.

4. Build a Docker\* image:

  ```bash
  docker image build -t ie_cross_armhf ie_cross_armhf
  ```

5. Run Docker\* container with mounted source code folder from host:

  ```bash
  docker run -it -v /absolute/path/to/openvino:/openvino ie_cross_armhf /bin/bash
  ```

6. While in the container:

    1. Go to the cloned `openvino` repository:

      ```bash
      cd openvino
      ```

    2. Create a build folder:

      ```bash
      mkdir build && cd build
      ```

    3. Build the Inference Engine:

      ```bash
      cmake -DCMAKE_BUILD_TYPE=Release \
          -DCMAKE_TOOLCHAIN_FILE="../cmake/arm.toolchain.cmake" \
          -DTHREADS_PTHREAD_ARG="-pthread" .. && \
          make --jobs=$(nproc --all)
      ```

7. Press **Ctrl+D** to exit from Docker. You can find the resulting binaries
   in the `openvino/bin/armv7l/` directory and the OpenCV*
   installation in the `openvino/inference-engine/temp`.

>**NOTE**: Native applications that link to cross-compiled Inference Engine
library require an extra compilation flag `-march=armv7-a`.

### Additional Build Options

You can use the following additional build options:

- Required versions of OpenCV packages are downloaded automatically by the
  CMake-based script. If you want to use the automatically downloaded packages
  but you already have installed OpenCV packages configured in your environment,
  you may need to clean the `OpenCV_DIR` environment variable before running
  the `cmake` command; otherwise they won't be downloaded and the build may
  fail if incompatible versions were installed.

- If the CMake-based build script cannot find and download the OpenCV package
  that is supported on your platform, or if you want to use a custom build of
  the OpenCV library, see: [Use Custom OpenCV Builds](#use-custom-opencv-builds-for-inference-engine)
  for details.

- To build Python API wrapper, install `libpython3-dev:armhf` and `python3-pip`
  packages using `apt-get`; then install `numpy` and `cython` python modules
  via `pip3`, adding the following options:
   ```sh
   -DENABLE_PYTHON=ON \
   -DPYTHON_EXECUTABLE=/usr/bin/python3.5 \
   -DPYTHON_LIBRARY=/usr/lib/arm-linux-gnueabihf/libpython3.5m.so \
   -DPYTHON_INCLUDE_DIR=/usr/include/python3.5
   ```

- nGraph-specific compilation options:
  `-DNGRAPH_ONNX_IMPORT_ENABLE=ON` enables the building of the nGraph ONNX importer.
  `-DNGRAPH_DEBUG_ENABLE=ON` enables additional debug prints.

### (Optional) Additional Installation Steps for the Intel® Neural Compute Stick 2

> **NOTE**: These steps are only required if you want to perform inference on the
Intel® Neural Compute Stick 2 using the Inference Engine MYRIAD Plugin. See also
[Intel® Neural Compute Stick 2 Get Started].

#### For Linux, Raspbian\* Stretch OS 

1. Add the current Linux user to the `users` group; you will need to log out and
   log in for it to take effect:
```sh
sudo usermod -a -G users "$(whoami)"
```

2. To perform inference on Intel® Neural Compute Stick 2, install the USB rules
as follows:
```sh
cat <<EOF > 97-myriad-usbboot.rules
SUBSYSTEM=="usb", ATTRS{idProduct}=="2485", ATTRS{idVendor}=="03e7", GROUP="users", MODE="0666", ENV{ID_MM_DEVICE_IGNORE}="1"
SUBSYSTEM=="usb", ATTRS{idProduct}=="f63b", ATTRS{idVendor}=="03e7", GROUP="users", MODE="0666", ENV{ID_MM_DEVICE_IGNORE}="1"
EOF
```
```sh
sudo cp 97-myriad-usbboot.rules /etc/udev/rules.d/
```
```sh
sudo udevadm control --reload-rules
```
```sh
sudo udevadm trigger
```
```sh
sudo ldconfig
```
```sh
rm 97-myriad-usbboot.rules
```


[Intel® Neural Compute Stick 2 Get Started]:https://software.intel.com/en-us/neural-compute-stick/get-started