# Build on macOS* Systems

> **NOTE**: The current version of the OpenVINO™ toolkit for macOS* supports
inference on Intel CPUs only.

The software was validated on:
- macOS\* 10.15, 64-bit
- macOS\* 11.5, 64-bit

## Table of content

  - [Software Requirements](#software-requirements)
  - [Build Steps](#build-steps)
  - [Additional Build Options](#additional-build-options)

### Software Requirements

- [CMake]\* 3.13 or higher
- Clang\* compiler from Xcode\* 10.1 or higher
- Python\* 3.6 or higher for the Inference Engine Python API wrapper
- libusb library (e.g., **brew install libusb**)

### Build Steps

1. Clone submodules:
    ```sh
    cd openvino
    git submodule update --init --recursive
    ```
2. Create a build folder:
```sh
  mkdir build && cd build
```
3. Inference Engine uses a CMake-based build system. In the created `build`
   directory, run `cmake` to fetch project dependencies and create Unix makefiles,
   then run `make` to build the project:
```sh
  cmake -DCMAKE_BUILD_TYPE=Release ..
  make --jobs=$(nproc --all)
```
### Additional Build Options

You can use the following additional build options:

- Internal JIT GEMM implementation is used by default.

- Threading Building Blocks (TBB) is used by default. To build the Inference
  Engine with OpenMP* threading, set the `-DTHREADING=OMP` option.

- Required versions of TBB and OpenCV packages are downloaded automatically by
  the CMake-based script. If you want to use the automatically downloaded
  packages but you already have installed TBB or OpenCV packages configured in
  your environment, you may need to clean the `TBBROOT` and `OpenCV_DIR`
  environment variables before running the `cmake` command, otherwise they won't
  be downloaded and the build may fail if incompatible versions were installed.

- If the CMake-based build script can not find and download the OpenCV package
  that is supported on your platform, or if you want to use a custom build of
  the OpenCV library, refer to the
  [Use Custom OpenCV Builds](#use-custom-opencv-builds-for-inference-engine)
  section for details.

- To build the Python API wrapper, you must enable the `-DENABLE_PYTHON=ON` option. To
  specify an exact Python version, use the following suggested options:
   - If you installed Python through Homebrew* (recommended), please first install the following libraries and dependencies.
   ```sh
   pip3 install clang==9.0
   pip3 install cython
   pip3 install pyyaml
   ```
   - Then, you can enable Python API Wrapper with the option enabled (please refer to step # 3 above). 
   ```
   cmake -DCMAKE_BUILD_TYPE=Release -DENABLE_PYTHON=ON \
   -DPYTHON_EXECUTABLE=/usr/local/Cellar/python@3.7/3.7.11/Frameworks/Python.framework/Versions/3.7/bin/python3.7m \
   -DPYTHON_LIBRARY=/usr/local/Cellar/python@3.7/3.7.11/Frameworks/Python.framework/Versions/3.7/lib/libpython3.7m.dylib \ 
   -DPYTHON_INCLUDE_DIR=/usr/local/Cellar/python@3.7/3.7.11/Frameworks/Python.framework/Versions/3.7/include/python3.7m ..
   ```
   - If you installed Python other ways, you can use the following commands to find where the `dylib` and `include_dir` are located, respectively, and update the option parameters above accordingly:
   ```sh
   find /usr/ -name 'libpython*m.dylib'
   find /usr/ -type d -name python3.7m
   ```
- nGraph-specific compilation options:
  `-DNGRAPH_ONNX_IMPORT_ENABLE=ON` enables the building of the nGraph ONNX importer.
  `-DNGRAPH_DEBUG_ENABLE=ON` enables additional debug prints.


[CMake]:https://cmake.org/download/
