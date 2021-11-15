# CMake Options for Custom Compilation

This document provides description and default values for CMake options that can be used to build a custom OpenVINO runtime using the open source version. For instructions on how to create a custom runtime from the prebuilt OpenVINO release package, refer to the [deployment manager] documentation. To understand all the dependencies when creating a custom runtime from the open source repository, refer to the [Inference Engine Introduction].

## Table of contents:

* [Disable / enable plugins build and other components](#disable--enable-plugins-build-and-other-components)
* [Options affecting binary size](#options-affecting-binary-size)
* [Test capabilities](#test-capabilities)
* [Other options](#other-options)

## Disable / enable plugins build and other components

* Inference backends:
    * `ENABLE_MKL_DNN` enables CPU plugin compilation:
        * `ON` is default for x86 platforms; `OFF`, otherwise.
    * `ENABLE_CLDNN` enables GPU plugin compilation:
        * `ON` is default for x86 platforms; not available, otherwise.
    * `ENABLE_GNA` enables GNA plugin compilation:
        * `ON` is default for x86 platforms; not available, otherwise.
    * `ENABLE_VPU` enables VPU (Myriad and HDDL only) components build:
        * `ON` is default.
    * `ENABLE_MYRIAD` enables MYRIAD plugin build:
        * `ON` is default.
    * `ENABLE_HETERO` enables HETERO plugin build:
        * `ON` is default.
    * `ENABLE_MULTI` enables AUTO / MULTI plugin build:
        * `ON` is default.
    * `ENABLE_TEMPLATE` enables TEMPLATE plugin build:
        * `ON` is default.
* Frontends to work with models from frameworks:
    * `NGRAPH_ONNX_FRONTEND_ENABLE` enables [ONNX] frontend plugin for OpenVINO Runtime:
        * `ON` is default.
    * `NGRAPH_PDPD_FRONTEND_ENABLE` enables [PDPD] frontend plugin for OpenVINO Runtime:
        * `ON` is default.
    * `NGRAPH_TF_FRONTEND_ENABLE` enables [TensorFlow] frontend plugin for OpenVINO Runtime:
        * `ON` is default.
    * `NGRAPH_IR_FRONTEND_ENABLE` enables OpenVINO Intermediate Representation frontend plugin for OpenVINO Runtime:
        * `ON` is default.
* `IE_EXTRA_MODULES` specifies path to add extra OpenVINO modules to the build.
    * See [OpenVINO Contrib] to add extra modules from.
* `ENABLE_SAMPLES` enables Inference Engine samples build:
    * `ON` is default.
* `ENABLE_PYTHON` enables [Python] API build:
    * `OFF` is default.
* `ENABLE_TESTS` enables tests compilation:
    * `OFF` is default.
* `ENABLE_IR_V7_READER` enables IR v7 reader:
    * `ON` is default.
    **Note:** must be turned `OFF` when building OpenVINO runtime as static
* `NGRAPH_UNIT_TEST_ENABLE` enables ngraph unit tests:
    * The value is the same as for the `ENABLE_TESTS` option.
* `ENABLE_DOCS` enables building the OpenVINO documentation:
    * `OFF` is default.
* `ENABLE_SYSTEM_PUGIXML` builds with system version of [pugixml] if it is available on the system.
    * `OFF` is default.
    * [Inference Engine thirdparty pugixml] is used by default.
* `NGRAPH_USE_SYSTEM_PROTOBUF` use [protobuf] installed on the system:
    * `OFF` is default.
    * Affects ONNX importer component only.

## Options affecting binary size

* `ENABLE_LTO` boolean option to enable [Link Time Optimizations]:
    * `OFF` is default, because it takes longer time to link binaries.
    * `ON` is enabled for OpenVINO release binaries.
    * Available on Unix* compilers only like GCC or CLANG.
    * Gives 30% decrease in binary size together with other optimization options used to build OpenVINO.
* `THREADING` points to the OpenVINO threading interface:
    * `TBB` is the default option, which enables build with [Intel TBB] and `tbb::static_partitioner`.
    * `TBB_AUTO` enables building with [Intel TBB].
    * `OMP` enables building with Intel OpenMP.
    * `SEQ` disables threading optimizations. Can be used in cases when TBB binaries are absent.
    * **Note:** because TBB is a template library, it increases binary size because of multiple instantiations of `tbb::parallel_for`
* `ENABLE_TBBBIND_2_5` enables prebuilt static TBBBind 2.5 usage:
    * `ON` is default, because OpenVINO Runtime should be generic out of box.

> **Note:** TBBBind 2.5 is needed when OpenVINO **inference** targets CPUs with:
> * NUMA support (Non-Unified Memory Architecture), e.g. to detect a number of NUMA nodes
> * Hybrid architecture to separate Performance/Efficiency cores and schedule tasks in the optimal way.

> **Note:** if you build OpenVINO runtime with [oneTBB] support where TBBBind 2.5 is automatically loaded by TBB in runtime, then set `ENABLE_TBBBIND_2_5` to `OFF`

* `ENABLE_SSE42` enables SSE4.2 optimizations:
    * `ON` is default for x86 platforms; not available for other platforms.
    * Affects only Inference Engine common part and preprocessing plugin, **does not affect the mkldnn library**
* `ENABLE_AVX2` enables AVX2 optimizations:
    * `ON` is default for x86 platforms, not available for other platforms.
    * Affects only Inference Engine common part and preprocessing plugin, **does not affect the mkldnn library**
* `ENABLE_AVX512F` enables AVX512 optimizations:
    * `ON` is default for x86 platforms, not available for other platforms.
    * Affects only Inference Engine common part and preprocessing plugin, **does not affect the mkldnn library**
* `ENABLE_PROFILING_ITT` enables profiling with [Intel ITT and VTune]. 
    * `OFF` is default, because it increases binary size.
* `SELECTIVE_BUILD` enables [[Conditional compilation|ConditionalCompilation]] feature.
    * `OFF` is default.
## Test capabilities

* `ENABLE_SANITIZER` builds with clang [address sanitizer] support:
    * `OFF` is default.
* `ENABLE_THREAD_SANITIZER` builds with clang [thread-sanitizer] support:
    * `OFF` is default.
* `ENABLE_COVERAGE` adds option to enable coverage. See dedicated guide [[how to measure test coverage|InferenceEngineTestsCoverage]]:
    * `OFF` is default.
* `ENABLE_FUZZING` enables instrumentation of code for fuzzing:
    * `OFF` is default.

## Other options

* `ENABLE_CPPLINT` enables code style check using [cpplint] static code checker:
    * `ON` is default.
* `ENABLE_CLANG_FORMAT` enables [Clang format] code style check:
    * `ON` is default.
    * Used only for ngraph component.
* `TREAT_WARNING_AS_ERROR` treats all warnings as an error:
    * `OFF` is default.
* `ENABLE_FASTER_BUILD` enables [precompiled headers] and [unity build] using CMake:
    * `OFF` is default.
* `ENABLE_INTEGRITYCHECK` builds DLLs with [/INTEGRITYCHECK] flag:
    * `OFF` is default.
    * Available on MSVC compiler only.

[Link Time Optimizations]:https://llvm.org/docs/LinkTimeOptimization.html
[thread-sanitizer]:https://clang.llvm.org/docs/ThreadSanitizer.html
[address sanitizer]:https://clang.llvm.org/docs/AddressSanitizer.html
[Intel ITT and VTune]:https://software.intel.com/content/www/us/en/develop/documentation/vtune-help/top/api-support/instrumentation-and-tracing-technology-apis.html
[precompiled headers]:https://cmake.org/cmake/help/git-stage/command/target_precompile_headers.html
[unity build]:https://cmake.org/cmake/help/latest/prop_tgt/UNITY_BUILD.html
[/INTEGRITYCHECK]:https://docs.microsoft.com/en-us/cpp/build/reference/integritycheck-require-signature-check?view=msvc-160
[Intel TBB]:https://software.intel.com/content/www/us/en/develop/tools/threading-building-blocks.html
[Python]:https://www.python.org/
[Java]:https://www.java.com/ru/
[cpplint]:https://github.com/cpplint/cpplint
[Clang format]:http://clang.llvm.org/docs/ClangFormat.html
[OpenVINO Contrib]:https://github.com/openvinotoolkit/openvino_contrib
[Inference Engine thirdparty pugixml]:https://github.com/openvinotoolkit/openvino/tree/master/inference-engine/thirdparty/pugixml
[pugixml]:https://pugixml.org/
[ONNX]:https://onnx.ai/
[protobuf]:https://github.com/protocolbuffers/protobuf
[deployment manager]:https://docs.openvinotoolkit.org/latest/openvino_docs_install_guides_deployment_manager_tool.html
[Inference Engine Introduction]:https://docs.openvinotoolkit.org/latest/openvino_docs_IE_DG_inference_engine_intro.html
[PDPD]:https://github.com/PaddlePaddle/Paddle
[TensorFlow]:https://www.tensorflow.org/
[oneTBB]:https://github.com/oneapi-src/oneTBB