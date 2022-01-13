# GPU plugin structure

Historically GPU plugin was built on top of standalone [clDNN library](https://github.com/intel/clDNN) for DNNs inference on Intel® GPUs,
but at some point clDNN became a part of OpenVINO, so now it's a part of overall GPU plugin code.

OpenVINO GPU plugin is responsible for:
 1. [IE Plugin API](https://github.com/openvinotoolkit/openvino/blob/master/docs/IE_PLUGIN_DG/Intro.md) implementation.
 2. Translation of model from common IE semantic (ov::Function) into plugin specific one (cldnn::topology) which is then compiled into
 gpu graph representation (cldnn::network).
 3. Implementation of OpenVINO operation set for Intel® GPU.
 4. Device specific graph transformations.
 5. Memory allocation and management logic.
 6. Processing of incoming InferRequests using clDNN objects.
 7. Actual execution on GPU device.

As Intel GPU Plugin source code structure is shown below:
```
- src/plugins/intel_gpu - root GPU plugin folder
 - include/intel_gpu/ - library internal headers
   - graph/ - headers for internal graph representations
   - primitives/ - primitive definitions for all supported operations
   - runtime/ - abstraction for execution runtime entities (memory, device, engine, etc)
   - plugin/ - definition of classes required for OpenVINO plugin API implementation
 - src/
   - kernel_selector/ - OpenCL™ kernels (host+device parts) + utils for optimal kernels selection
     - common/ - definition of some generic classes/structures used in kernel_selector
     - core/ - kernels, kernel selectors, and kernel parameters definitions
       - actual_kernels/ - host side part of OpenCL™ kernels including applicability checks, performance heuristics and Local/Global work-groups description
       - cache/
         - cache.json - tuning cache of the kernels which is redistributed with the plugin to improve kernels and kernel parameters selection for better performance
       - cl_kernels/ - templates of GPU kernels (device part) written on OpenCL™
       - common/ - utils for code generation and kernels selection
   - plugin/ - implementation of OpenVINO plugin API
     - ops/ - factories for conversion of OpenVINO operations to internal primitives
   - graph/ - all sources related to internal graph representation
     - impls/ - definition of primitive implementations
     - graph_optimizer/ - passes for graph transformations
     - include/ - headers with graph nodes
   - runtime/ - static library with runtime implementation
      - ocl/ - implementation for OpenCL™ based runtime
 - tests/ - unit tests
 - thirdparty/
   - rapidjson/ - thirdparty [RapidJSON](https://github.com/Tencent/rapidjson) lib for reading json files (cache.json)
   - onednn_gpu/ - [oneDNN](https://github.com/oneapi-src/oneDNN) submodule which may be used to accelerate some primitives
```

One last thing that is worth mentioning is functional tests which is located in the following location:
```
src/tests/functional/plugin/gpu
```
Most of the tests are reused across plugins, and each plugin only need to add test instances with some specific parameters.

Shared tests are located here:
```
src/tests/functional/plugin/shared                        <--- test definitions
src/tests/functional/plugin/gpu/shared_tests_instances    <--- instances for GPU plugin
```
