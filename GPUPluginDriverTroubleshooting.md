# Driver issues troubleshooting

If you see errors like "[CLDNN ERROR]. clGetPlatformIDs error -1001" when running OpenVINO samples / demoes, then most likely you have some issues with OpenCL runtime on your machine. This document contains several hints on what to check and how to troubleshoot such kind of issues.

## 1. Make sure that you have GPU on your system
Some Intel® CPUs might not have integrated GPU, so if you want to run OpenVINO on iGPU, go to https://ark.intel.com/ and make sure that your CPU has it

## 2. Make sure that OpenCL® Runtime is installed
On Windows OpenCL runtime is a part of the GPU driver, but on linux it should be installed separately. For the installation tips please refer to [OpenVINO docs](https://docs.openvinotoolkit.org/latest/openvino_docs_install_guides_installing_openvino_linux.html) and [OpenCL Compute Runtime docs](https://github.com/intel/compute-runtime/blob/master/opencl/doc/DISTRIBUTIONS.md)

In order to make sure that OpenCL runtime is functional on your machine, you can use [clinfo](https://github.com/Oblomov/clinfo) tool. On many linux distributives it can be installed via package manager. If it's not available for your system, it can be easily built from sources.

## 3. Make sure that user has all required permissions to work with GPU device
TBD

## 4. Make sure that iGPU is enabled
TBD

## 5. Make sure that "/etc/OpenCL/vendors/intel.icd" contain proper paths to the OpenCL driver
TBD

## 6. Use LD_DEBUG=libs to trace loaded libraries
More details: https://github.com/bashbaug/OpenCLPapers/blob/markdown/OpenCLOnLinux.md







