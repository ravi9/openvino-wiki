# Building OpenVINO static libraries

## Contents

- [Introduction](#introduction)
- [Configure OpenVINO runtime in CMake stage](#configure-openvino-runtime-in-cmake-stage)
- [Build static OpenVINO libraries](#build-static-openvino-libraries)
- [Link static OpenVINO runtime](#link-static-openvino-runtime)
- [Static OpenVINO libraries + Conditional compilation for particular models](#static-openvino-libraries--conditional-compilation-for-particular-models)
- [Limitations](#limitations)

## Introduction

Building static OpenVINO Runtime libraries allows to additionally reduce a binary size when it's used together with conditional compilation.
This happens because not all interface symbols of OpenVINO runtime libraries are exported to end users during static build, which allows linker to remove them. See [Static OpenVINO libraries + Conditional compilation for particular models](#static-openvino-libraries--conditional-compilation-for-particular-models)

## Configure OpenVINO runtime in CMake stage

Default architecture of OpenVINO runtime assumes that the following components are subject to dynamic loading during execution time:
* (Device) Inference backends (CPU, GPU, MULTI, HETERO, etc)
* (Model) Frontends (IR, ONNX, PDPD, etc)
* Preprocessing library (to perform preprocessing like resize, color space conversions)
* IR v7 reader (used in legacy tests only, if you are not to going to run OpenVINO tests, set `-DENABLE_TESTS=OFF` which disables IR v7 reader)

With the static OpenVINO runtime, all these modules should be linked into a final user application and a **list of the modules/configuration must be known on CMake configure stage**, so to minimize the total binary size, explicitly turn `OFF` unnecessary components. Use [[CMake Options for Custom Compilation|CMakeOptionsForCustomCompilation ]] as a reference for OpenVINO CMake configuration.

For example, to enable only IR v11 reading and CPU inference capabilities without G-API preprocessing, use:
```sh
cmake -DENABLE_VPU=OFF \
      -DENABLE_CLDNN=OFF \
      -DENABLE_GNA=OFF \
      -DENABLE_MYRIAD=OFF \
      -DENABLE_HETERO=OFF \
      -DENABLE_MULTI=OFF \
      -DENABLE_TEMPLATE=OFF \
      -DENABLE_IR_V7_READER=OFF \
      -DNGRAPH_ONNX_FRONTEND_ENABLE=OFF \
      -DNGRAPH_PDPD_FRONTEND_ENABLE=OFF \
      -DNGRAPH_TF_FRONTEND_ENABLE=OFF \
      -DENABLE_GAPI_PREPROCESSING=OFF \
      -DENABLE_MKL_DNN=ON \
      -DNGRAPH_IR_FRONTEND_ENABLE=ON
```

**Note:** Inference backends located in external repositories can also be used in static build. Use `-DIE_EXTRA_MODULES=<path to external plugin root>` to enable them. `InferenceEngineDeveloperPackage.cmake` must not be used to build external plugins, only `IE_EXTRA_MODULES` is working.

**Note:** the `ENABLE_LTO` CMake option can also be passed to enable link time optimizations to reduce binary size. But such property should also be enabled on target which links with static OpenVINO libraries via `
set_target_properties(<target_name> PROPERTIES INTERPROCEDURAL_OPTIMIZATION_RELEASE ON)`

## Build static OpenVINO libraries

To build OpenVINO runtime in a static mode, you need to specify the additional CMake option:

```sh
cmake -DBUILD_SHARED_LIBS=OFF <all other CMake options> <openvino_sources root>
```

Then, use an usual CMake 'build' command:

```sh
cmake --build . --target inference_engine --config Release -j12
```

Then, the installation step:

```sh
cmake -DCMAKE_INSTALL_PREFIX=<install_root> -P cmake_install.cmake
```

The OpenVINO runtime is located in `<install_root>/runtime/lib`

## Link static OpenVINO runtime

Once you build static OpenVINO Runtime libraries and installed them, you can use one of the ways to add them to your project

### CMake interface

Just use CMake's `find_package` as usual and link `openvino::runtime`:

```cmake
find_package(OpenVINO REQUIRED)
target_link_libraries(<application> PRIVATE openvino::runtime)
```

`openvino::runtime` transitively adds all other static OpenVINO libraries to linker command. 

### Pass libraries to linker directly

If you want to configure your project directly, you need to pass all libraries from `<install_root>/runtime/lib` to linker command.

**Note:** since the proper order of static libraries must be used (dependent library should come **before** dependency in a linker command), suggestion is to use the following compiler specific flags to link static OpenVINO libraries:

Microsoft Visual Studio compiler:
```sh
/WHOLEARCHIVE:<ov_library 0> /WHOLEARCHIVE:<ov_library 1> ...
```

GCC like compiler:
```sh
gcc main.cpp -Wl,--whole-archive <all libraries from <root>/runtime/lib> > -Wl,--no-whole-archive -o a.out
```

## Static OpenVINO libraries + Conditional compilation for particular models

OpenVINO Runtime can be compiled for particular models using [[Conditional compilation for particular models|ConditionalCompilation]] guide.
Conditional compilation feature can be paired with static OpenVINO libraries to build even smaller end user applications in terms of binary size. The following procedure can be used (based on detailed [[Conditional compilation for particular models|ConditionalCompilation]] guide):

* Build OpenVINO Runtime as usual with CMake option `-DSELECTIVE_BUILD=COLLECT`
* Run targets applications on targets models and target platforms to collect traces
* Build final OpenVINO static Runtime with `-DSELECTIVE_BUILD=ON -DSELECTIVE_BUILD_STAT=/path/*.csv -DBUILD_SHARED_LIBS=OFF`

## Limitations

* CMake version 3.17 or higher must be used to build static OpenVINO libraries.
* Supported OSes:
    * Windows x64
    * Linux x64
    * All other OSes may work, but not explicitly tested
* Enabled and tested capabilities on OpenVINO runtime in static build:
    * OpenVINO common runtime - work with `ov::Function`, perform model loading on particular device
    * CPU, MULTI, HETERO, AUTO inference plugins
    * **Not enabled:** GNA, GPU, MYRIAD
    * **Still compiled as shared libraries**: IR, ONNX, PDPD and TF frontends to read `ov::Function`
* Static build support building of static libraries only for OpenVINO Runtime libraries. All other thirdparty prebuilt dependencies remain in the same format:
    * `libGNA` is a shared library
    * `TBB` is a shared library; to provide your own TBB build from [[oneTBB source code|https://github.com/oneapi-src/oneTBB]] use `export TBBROOT=<tbb_root>` before OpenVINO CMake scripts are run

    **Note:** TBB team does not recommend to use oneTBB as a static library, see [[Why onetbb not like static library?|https://github.com/oneapi-src/oneTBB/issues/646]]

* `TBBBind_2_5` is not available on Windows x64 during static OpenVINO build (see description for `ENABLE_TBBBIND_2_5` CMake option [[here|CMakeOptionsForCustomCompilation]] to understand what this library is responsible for). So, capabilities enabled by `TBBBind_2_5` are not available. To enable these capabilities, build [[oneTBB from source code|https://github.com/oneapi-src/oneTBB]] and provide path to built oneTBB artifacts via `TBBROOT` environment variable before OpenVINO CMake scripts are run.

* `ov::Op::type_info` static member is deprecated and not available in static build. Don't use `type_info` during implementation of your own custom operations, use `ov::Op::get_type_info_static()` instead 