# Build on Linux\* Systems

The software was validated on:
- Ubuntu\* 16.04 (64-bit) with default GCC\* 5.4.0
- Ubuntu\* 18.04 (64-bit) with default GCC\* 7.5.0
- Ubuntu\* 20.04 (64-bit) with default GCC\* 9.3.0
- CentOS\* 7.6 (64-bit) with default GCC\* 4.8.5

## Table of content:

  - [Software Requirements](#software-requirements)
  - [Build Steps](#build-steps)
  - [Additional Build Options](#additional-build-options)

### Software Requirements

- [CMake]\* 3.13 or higher
- GCC\* 4.8 or higher to build the Inference Engine
- Python 3.6 or higher for Inference Engine Python API wrapper
- (Optional) [Install Intel® Graphics Compute Runtime for OpenCL™ Driver package 19.41.14441].

### Build Steps
1. Clone submodules:
    ```sh
    cd openvino
    git submodule update --init --recursive
    ```
2. Install build dependencies using the `install_build_dependencies.sh` script in the
   project root folder.
   ```sh
   chmod +x install_build_dependencies.sh
   ```
   ```sh
   ./install_build_dependencies.sh
   ```
3. By default, the build enables the Inference Engine GPU plugin to infer models
   on your Intel® Processor Graphics. This requires you to
   [Install Intel® Graphics Compute Runtime for OpenCL™ Driver package 19.41.14441]
   before running the build. If you don't want to use the GPU plugin, use the
   `-DENABLE_CLDNN=OFF` CMake build option and skip the installation of the
   Intel® Graphics Compute Runtime for OpenCL™ Driver.
4. Create a build folder:
```sh
  mkdir build && cd build
```
5. Inference Engine uses a CMake-based build system. In the created `build`
   directory, run `cmake` to fetch project dependencies and create Unix
   makefiles, then run `make` to build the project:
```sh
  cmake -DCMAKE_BUILD_TYPE=Release ..
  make --jobs=$(nproc --all)
```

### Additional Build Options

You can use the following additional build options:

- Threading Building Blocks (TBB) is used by default. To build the Inference
  Engine with OpenMP\* threading, set the `-DTHREADING=OMP` option.

- Required versions of TBB and OpenCV packages are downloaded automatically by
  the CMake-based script. If you want to use the automatically downloaded
  packages but you already have installed TBB or OpenCV packages configured in
  your environment, you may need to clean the `TBBROOT` and `OpenCV_DIR`
  environment variables before running the `cmake` command, otherwise they
  will not be downloaded and the build may fail if incompatible versions were
  installed.

- If the CMake-based build script can not find and download the OpenCV package
  that is supported on your platform, or if you want to use a custom build of
  the OpenCV library, refer to the
  [Use Custom OpenCV Builds](#use-custom-opencv-builds-for-inference-engine)
  section for details.

- To build the Python API wrapper:
  1. Install all additional packages listed in the
     `/inference-engine/ie_bridges/python/src/requirements-dev.txt` file:
     ```sh
     pip install -r requirements-dev.txt
     ```
  2. Use the `-DENABLE_PYTHON=ON` option. To specify an exact Python version, use the following
     options:
     ```
     -DPYTHON_EXECUTABLE=`which python3.7` \
     -DPYTHON_LIBRARY=/usr/lib/x86_64-linux-gnu/libpython3.7m.so \
     -DPYTHON_INCLUDE_DIR=/usr/include/python3.7
     ```

- To switch the CPU and GPU plugins off/on, use the `cmake` options
  `-DENABLE_MKL_DNN=ON/OFF` and `-DENABLE_CLDNN=ON/OFF` respectively.

- nGraph-specific compilation options:
  `-DNGRAPH_ONNX_IMPORT_ENABLE=ON` enables the building of the nGraph ONNX importer.
  `-DNGRAPH_DEBUG_ENABLE=ON` enables additional debug prints.


[CMake]:https://cmake.org/download/
[Install Intel® Graphics Compute Runtime for OpenCL™ Driver package 19.41.14441]:https://github.com/intel/compute-runtime/releases/tag/19.41.14441
[MKL-DNN repository]:https://github.com/intel/mkl-dnn/releases/download/v0.19/mklml_lnx_2019.0.5.20190502.tgz