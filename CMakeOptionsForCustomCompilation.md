# CMake options for custom compilation

## Table of content:

* [Disable / enable plugins build and other components](#disable--enable-plugins-build-and-other-components)
* [Options affecting binary size](#options-affecting-binary-size)
* [Test capabilities](#test-capabilities)
* [Other options](#other-options)

## Disable / enable plugins build and other components

* `ENABLE_MKL_DNN` enables CPU plugin compilation:
    * `ON` is default for x86 platforms, `OFF` otherwise.
* `ENABLE_CLDNN` enables GPU plugin compilation:
    * `ON` is default for x86 platforms, not available otherwise.
* `ENABLE_GNA` enables GNA plugin compilation:
    * `ON` is default for x86 platforms, not available otherwise.
* `ENABLE_VPU` enables VPU (Myriad, HDDL) components build:
    * `ON` is default.
* `ENABLE_MYRIAD` enables MYRIAD plugin build:
    * `ON` is default.
* `NGRAPH_ONNX_IMPORT_ENABLE` enables [ONNX] importer plugin for Inference Engine:
    * `ON` is default.
* `ENABLE_SAMPLES` enables Inference Engine samples build:
    * `ON` is default.
* `ENABLE_PYTHON` enables [Python] API build:
    * `OFF` is default.
* `ENABLE_JAVA` enables [Java] API build:
    * `OFF` is default.
* `ENABLE_TESTS` enables tests compilation:
    * `OFF` is default.
* `NGRAPH_UNIT_TEST_ENABLE` enables ngraph unit tests:
    * The value is the same as for the option `ENABLE_TESTS`.
* `ENABLE_DOCS` enables build of OpenVINO documentation:
    * `OFF` is default.
* `IE_EXTRA_MODULES` specifies path to add extra OpenVINO modules to the build.
    * See [OpenVINO Contrib] to add extra modules from.
* `USE_SYSTEM_PUGIXML` builds with system version of [pugixml] if it's available on the system.
    * `OFF` is default.
    * [Inference Engine thirdparty pugixml] is used by default.
* `NGRAPH_USE_PROTOBUF_LITE` compiles and links with [protobuf] lite:
    * `OFF` is default.
    * Affects ONNX importer component only.
* `NGRAPH_USE_SYSTEM_PROTOBUF` use [protobuf] installed on the system:
    * `OFF` is default.
    * Affects ONNX importer component only.

## Options affecting binary size

* `ENABLE_LTO` boolean option to enable [Link Time Optimizations]:
    * `OFF` is default, because it takes longer time to link binaries.
    * `ON` is enabled for OpenVINO release binaries
    * Available on Unix compilers only like GCC or CLANG.
    * Gives 30% decrease in binary size together with other optimizations option using to build OpenVINO.
* `THREADING` points to OpenVINO threading interface:
    * `TBB` is a default option, enables build with [Intel TBB] and `tbb::static_partitioner`.
    * `TBB_AUTO` enables build with [Intel TBB].
    * `OMP` enables build with Intel OpenMP.
    * `SEQ` disable threading optimizations. Can be used in cases when TBB binaries are absent.
    * **Note:** because TBB is a template library, it increase binary size because of multiple instantiations of `tbb::parallel_for`
* `ENABLE_SSE42` enables SSE4.2 optimizations:
    * `ON` is default for x86 platforms, not available for other platforms.
    * Affects only Inference Engine common part and preprocessing plugin, **does not affect mkldnn library**
* `ENABLE_AVX2` enables AVX2 optimizations:
    * `ON` is default for x86 platforms, not available for other platforms.
    * Affects only Inference Engine common part and preprocessing plugin, **does not affect mkldnn library**
* `ENABLE_AVX512F` enables AVX512 optimizations:
    * `ON` is default for x86 platforms, not available for other platforms.
    * Affects only Inference Engine common part and preprocessing plugin, **does not affect mkldnn library**
* `ENABLE_PROFILING_ITT` enables profiling with [Intel ITT and VTune]. Default is `OFF`, because it increases binary size.

## Test capabilities

* `ENABLE_SANITIZER` builds with clang [address sanitizer] support:
    * `OFF` is default.
* `ENABLE_THREAD_SANITIZER` builds with clang [thread-sanitizer] support:
    * `OFF` is default.
* `COVERAGE` adds option to enable coverage. See dedicated guide [[how to measure test coverage|InferenceEngineTestsCoverage]]:
    * `OFF` is default.
* `ENABLE_FUZZING` enables instrumentation of code for fuzzing:
    * `OFF` is default.

## Other options

* `ENABLE_CPPLINT` enables code style check using [cpplint] static code checker:
    * `ON` is default.
* `ENABLE_CLANG_FORMAT` enables [Clang format] code style check:
    * `ON` is default.
    * Used only for ngraph component.
* `TREAT_WARNING_AS_ERROR` treats all warnings as error:
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