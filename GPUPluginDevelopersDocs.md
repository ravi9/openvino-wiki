# GPU plugin developers docs

GPU plugin in [OpenVINO toolkit](https://github.com/openvinotoolkit/openvino) supports inference on Intel® GPUs starting from Gen8 architecture.

- [[Source code structure|GPUPluginStructure]]
    - [[Basic data structures of gpu graph and overall flow]]
    - [[Memory allocation in GPU plugin]]
- [[Simplified workflow|GPUPluginWorkFlow]]
    - [[Graph Optimization Passes]]
    - [[Execution of Inference]]
- [[Memory formats|GPUPluginMemoryFormats]]
- [[Kernels and kernel selectors|GPUPluginKernels]]
- [[GPU plugin operations enabling flow|GPUPluginOpsEnabling]]
- [[Debug utils|GPUPluginDebugUtils]]
- [[OpenCL Runtime issues troubleshooting|GPUPluginDriverTroubleshooting]]
- [[GPU plugin unit test|GPU plugin unit test]]