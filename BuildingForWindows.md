# Build OpenVINO™ OpenVINO™ Runtime for Windows* systems

The software was validated on:
- Microsoft\* Windows\* 10 (64-bit) with Visual Studio 2019

## Table of content:

  - [Software Requirements](#software-requirements)
  - [Build Steps](#build-steps)
  - [Additional Build Options](#additional-build-options)
  - [Building Inference Engine with Ninja* Build System](#building-inference-engine-with-ninja-build-system)

### Software Requirements
- [CMake]\*3.13 or higher
- Microsoft\* Visual Studio 2019, version 16.8 or later
- (Optional) Intel® Graphics Driver for Windows* (26.20) [driver package].
- Python 3.6 or higher for OpenVINO Runtime Python API
- [Git for Windows*]

### Build Steps

1. Clone submodules:
    ```sh
    git submodule update --init --recursive
    ```
2. By default, the build enables the Inference Engine GPU plugin to infer models
   on your Intel® Processor Graphics. This requires you to download and install
   the Intel® Graphics Driver for Windows (26.20) [driver package] before
   running the build. If you don't want to use the GPU plugin, use the
   `-DENABLE_INTEL_GPU=OFF` CMake build option and skip the installation of the
   Intel® Graphics Driver.
3. Create build directory:
    ```sh
    mkdir build
    ```
4. In the `build` directory, run `cmake` to fetch project dependencies and
   generate a Visual Studio solution.

   For Microsoft\* Visual Studio 2019 x64 architecture:
    ```sh
    cmake -G "Visual Studio 16 2019" -A x64 -DCMAKE_BUILD_TYPE=Release ..
    ```

   For Microsoft\* Visual Studio 2019 ARM architecture:
    ```sh
    cmake -G "Visual Studio 16 2019" -A ARM -DCMAKE_BUILD_TYPE=Release ..
    ```

   For Microsoft\* Visual Studio 2019 ARM64 architecture:
    ```sh
    cmake -G "Visual Studio 16 2019" -A ARM64 -DCMAKE_BUILD_TYPE=Release ..
    ```

5. Build generated solution in Visual Studio or run
   `cmake --build . --config Release --verbose -j8` to build from the command line.

6. Before running the samples, add paths to the TBB and OpenCV binaries used for
   the build to the `%PATH%` environment variable. By default, TBB binaries are
   downloaded by the CMake-based script to the `<openvino_repo>/inference-engine/temp/tbb/bin`
   folder, OpenCV binaries to the `<openvino_repo>/inference-engine/temp/opencv_4.5.0/opencv/bin`
   folder.

### Additional Build Options

- Internal JIT GEMM implementation is used by default.

- Threading Building Blocks (TBB) is used by default. To build the Inference
  Engine with OpenMP* threading, set the `-DTHREADING=OMP` option.

- Required versions of TBB and OpenCV packages are downloaded automatically by
  the CMake-based script. If you want to use the automatically-downloaded
  packages but you already have installed TBB or OpenCV packages configured in
  your environment, you may need to clean the `TBBROOT` and `OpenCV_DIR`
  environment variables before running the `cmake` command; otherwise they won't
  be downloaded and the build may fail if incompatible versions were installed.

- If the CMake-based build script can not find and download the OpenCV package
  that is supported on your platform, or if you want to use a custom build of
  the OpenCV library, refer to the [Use Custom OpenCV Builds](#use-custom-opencv-builds-for-inference-engine)
  section for details.

- To switch off/on the CPU and GPU plugins, use the `cmake` options
  `-DENABLE_INTEL_CPU=ON/OFF` and `-DENABLE_INTEL_GPU=ON/OFF` respectively.

- To build the OpenVINO Runtime Python API:
   1. First, install all additional packages (e.g., cython and opencv) listed in the
     `src\bindings\python\src\compatibility\openvino\requirements-dev.txt` file:
      ```sh
      pip install -r requirements-dev.txt
      ```
  2. Second, enable the `-DENABLE_PYTHON=ON` in the CMake (Step #4) option above. To
  specify an exact Python version, use the following options:
     ```sh
     -DPYTHON_EXECUTABLE="C:\Program Files\Python37\python.exe" ^
     -DPYTHON_LIBRARY="C:\Program Files\Python37\libs\python37.lib" ^
     -DPYTHON_INCLUDE_DIR="C:\Program Files\Python37\include"
     ```

- OpenVINO runtime compilation options:
  `-DENABLE_OV_ONNX_FRONTEND=ON` enables the building of the ONNX importer.

### Building Inference Engine with Ninja* Build System

```sh
call "C:\Program Files (x86)\Microsoft Visual Studio\2019\Professional\VC\Auxiliary\Build\vcvars64.bat"
cmake -G Ninja -Wno-dev -DCMAKE_BUILD_TYPE=Release ..
cmake --build . --config Release
```


[CMake]:https://cmake.org/download/
[MKL-DNN repository for Windows]:(https://github.com/intel/mkl-dnn/releases/download/v0.19/mklml_win_2019.0.5.20190502.zip)
[OpenBLAS]:https://sourceforge.net/projects/openblas/files/v0.2.14/OpenBLAS-v0.2.14-Win64-int64.zip/download
[mingw64\* runtime dependencies]:https://sourceforge.net/projects/openblas/files/v0.2.14/mingw64_dll.zip/download
[driver package]:https://downloadcenter.intel.com/download/29335/Intel-Graphics-Windows-10-DCH-Drivers
[Git for Windows*]:https://gitforwindows.org/