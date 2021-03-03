# Conditional compilation for particular models

## Contents

- [Introduction](#introduction)
- [Building for different models](#building-for-different-models)
- [Building for devices with different ISA](#building-for-different-isa)

## Introduction

The conditional compilation lets significantly reduce OpenVINO™ binaries size
by excluding unnecessary components for particular models inference.
A short list of components that can be excluded from the result build are following:
* layers and graph transformations in nGraph and plugins
* nGraph operations
* jit kernels in CPU plugin
* arbitrary code that is not used for particular model inference

While conditional compilation can reduce the libraries size, it has a significant drawback. **The OpenVINO will work only with a selected set of models and devices.**

In order to take advantage of the conditional compilation functionality, the following tools should be installed first:
* [[Python|https://www.python.org]]

## Building for different models

Conditional compilation has two stages:
* Collecting information about code usage.
* Building the result binaries without unused components or parts.

1. Code usage analysis
    1. Run CMake tool with options: `-DENABLE_PROFILING_ITT=ON -DSELECTIVE_BUILD=COLLECT`
    2. Select several models to be used in a specific application or target device
    3. Use the target `sea_itt_lib` in order to build ITT collector
    4. Run target application under the ITT collector for code usage analysis for each model. Statistics are generated in .csv format.  
`python thirdparty/itt_collector/runtool/sea_runtool.py --bindir ${OPENVINO_LIBRARY_DIR} -o ${MY_MODEL_RESULT} ! ./benchmark_app -niter 1 -nireq 1 -m ${MY_MODEL}.xml`
2. Final build
    1. Run CMake tool with options: `-DSELECTIVE_BUILD=ON -DSELECTIVE_BUILD_STAT=${ABSOLUTE_PATH_TO_STATISTICS_FILES}/*.csv`
    2. `cmake --build <cmake_build_directory>`

The "-niter 1 -nireq 1" flags are highly recommended for bencmark_app. Otherwise, the trace files will be very large.
If you are using an application other than benchmark_app, remember to limit the number of inference requests and iterations.

## Building for devices with different ISA

Building for devices with different ISA is quite similar to building for different models (see previous chapter).
The differences are only in the code usage analysis step. The analysis step should be performed on target devices and all CSV files with statistics should be copied to the build machine. These files will be used for the final build.

## Limitations

* At the current moment we don't support Ninja build system for the conditional compilation build.


