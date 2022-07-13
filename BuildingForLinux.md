# Build on Linux\* Systems

The software was validated on:
- Ubuntu\* 18.04 (64-bit) with default GCC\* 7.5.0
- Ubuntu\* 20.04 (64-bit) with default GCC\* 9.3.0
- Red Hat\* Enterprise Linux\* 8.2 (64-bit) with default GCC\* 8.5.0

## Table of content:

  - [Software Requirements](#software-requirements)
  - [Build Steps](#build-steps)
  - [Additional Build Options](#additional-build-options)

### Software Requirements

- [CMake]\* 3.13 or higher
- GCC\* 7.5 or higher to build OpenVINO Runtime
- Python 3.6 or higher for OpenVINO Runtime Python API
- (Optional) [Install Intel® Graphics Compute Runtime for OpenCL™ Driver package 19.41.14441] to allow the OpenVINO installation to work with GPUs.

### Build Steps
1. Clone submodules:
    ```sh
    git clone https://github.com/openvinotoolkit/openvino.git
    cd openvino
    git submodule update --init --recursive
    ```
    (Optional) For users in China, clone submodules via gitee mirrors:
    ```
    cd openvino
    chmod +x scripts/submodule_update_with_gitee.sh
    ./scripts/submodule_update_with_gitee.sh
    ```

2. Install build dependencies using the `install_build_dependencies.sh` script in the
   project root folder.
   ```sh
   chmod +x install_build_dependencies.sh
   ```
   ```sh
   ./install_build_dependencies.sh
   ```
   > **NOTE**: By default, the build enables the OpenVINO Runtime GPU plugin to infer models on your Intel® Processor Graphics. This requires you to [Install Intel® Graphics Compute Runtime for OpenCL™ Driver package 19.41.14441] before running the build. If you don't want to use the GPU plugin, use the `-DENABLE_INTEL_GPU=OFF` CMake build option and skip the installation of the Intel® Graphics Compute Runtime for OpenCL™ Driver.

3. Create a build folder:
```sh
  mkdir build && cd build
```
4. OpenVINO Runtime uses a CMake-based build system. In the created `build`
   directory, run `cmake` to fetch project dependencies and create Unix
   makefiles, then run `make` to build the project:
```sh
  cmake -DCMAKE_BUILD_TYPE=Release ..
  make --jobs=$(nproc --all)
```
The process may take some time to finish.

### Additional Build Options

You can use the following additional build options:

- Threading Building Blocks (TBB) is used by default. To build the OpenVINO 
  Runtime with OpenMP\* threading, set the `-DTHREADING=OMP` option.

- For IA32 operation systems, use [ia32.linux.toolchain.cmake](https://github.com/openvinotoolkit/openvino/blob/master/cmake/toolchains/ia32.linux.toolchain.cmake) CMake toolchain file:

   ```sh
   cmake -DCMAKE_TOOLCHAIN_FILE=<openvino_repo>/cmake/toolchains/ia32.linux.toolchain.cmake ..
   ```

- Required versions of TBB and OpenCV packages are downloaded automatically by
  the CMake-based script. If you want to use the automatically downloaded
  packages but you have already installed TBB or OpenCV packages configured in
  your environment, you may need to clean the `TBBROOT` and `OpenCV_DIR`
  environment variables before running the `cmake` command, otherwise they
  will not be downloaded and the build may fail if incompatible versions were
  installed.

- If the CMake-based build script can not find and download the OpenCV package
  that is supported on your platform, or if you want to use a custom build of
  the OpenCV library, see how to [Use Custom OpenCV Builds](https://github.com/openvinotoolkit/openvino/wiki/CMakeOptionsForCustomCompilation#Building-with-custom-OpenCV).

- To build the OpenVINO Runtime Python API:
  1. First, install all additional packages (e.g., cython and opencv) listed in the
     `/src/bindings/python/src/compatibility/openvino/requirements-dev.txt` file:
     ```sh
     pip install -r requirements_dev.txt
     ```
  2. Second, enable the `-DENABLE_PYTHON=ON` in the CMake (Step #5) option above. To specify an exact Python version, use the following
     options:
     ```
     -DPYTHON_EXECUTABLE=`which python3.7` \
     -DPYTHON_LIBRARY=/usr/lib/x86_64-linux-gnu/libpython3.7m.so \
     -DPYTHON_INCLUDE_DIR=/usr/include/python3.7
     ```
  3. After the build process finishes, export the newly built Python libraries to the user environment variables: 
     ```
     export PYTHONPATH=PYTHONPATH:~/openvino/bin/intel64/Release/lib/python_api/python3.7/
     export LD_LIBRARY_PATH=LD_LIBRARY_PATH:~/openvino/bin/intel64/Release/lib/
     ```

- To switch the CPU and GPU plugins off/on, use the `cmake` options
  `-DENABLE_INTEL_CPU=ON/OFF` and `-DENABLE_INTEL_GPU=ON/OFF` respectively.

- OpenVINO runtime compilation options:
  `-DENABLE_OV_ONNX_FRONTEND=ON` enables the building of the ONNX importer.


[CMake]:https://cmake.org/download/
[Install Intel® Graphics Compute Runtime for OpenCL™ Driver package 19.41.14441]:https://github.com/intel/compute-runtime/releases/tag/19.41.14441
[MKL-DNN repository]:https://github.com/intel/mkl-dnn/releases/download/v0.19/mklml_lnx_2019.0.5.20190502.tgz