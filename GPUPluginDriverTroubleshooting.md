# Driver issues troubleshooting

If you see errors like "[CLDNN ERROR]. clGetPlatformIDs error -1001" when running OpenVINO samples / demos, then most likely you have some issues with OpenCL runtime on your machine. This document contains several hints on what to check and how to troubleshoot such kind of issues.

In order to make sure that OpenCL runtime is functional on your machine, you can use [clinfo](https://github.com/Oblomov/clinfo) tool. On many linux distributives it can be installed via package manager. If it's not available for your system, it can be easily built from sources.

Example of clinfo output:
```
Number of platforms                               1
  Platform Name                                   Intel(R) OpenCL HD Graphics
  Platform Vendor                                 Intel(R) Corporation

  ...

  Platform Name                                   Intel(R) OpenCL HD Graphics
Number of devices                                 1
  Device Name                                     Intel(R) Graphics [0x3e92]
  Device Vendor                                   Intel(R) Corporation
  Device Vendor ID                                0x8086
  Device Version                                  OpenCL 3.0 NEO 
  Driver Version                                  20.49.0
  Device OpenCL C Version                         OpenCL C 3.0 
  Device Type                                     GPU
```
## 1. Make sure that you have GPU on your system
Some Intel® CPUs might not have integrated GPU, so if you want to run OpenVINO on iGPU, go to https://ark.intel.com/ and make sure that your CPU has it

## 2. Make sure that OpenCL® Runtime is installed
On Windows OpenCL runtime is a part of the GPU driver, but on linux it should be installed separately. For the installation tips please refer to [OpenVINO docs](https://docs.openvinotoolkit.org/latest/openvino_docs_install_guides_installing_openvino_linux.html) and [OpenCL Compute Runtime docs](https://github.com/intel/compute-runtime/blob/master/opencl/doc/DISTRIBUTIONS.md)



## 3. Make sure that user has all required permissions to work with GPU device
Add the current Linux user to the `video` group:
```
sudo usermod -a -G video "$(whoami)"
```

## 4. Make sure that iGPU is enabled
```
$ cat /sys/devices/pci0000\:00/0000\:00\:02.0/enable
1
```

## 5. Make sure that "/etc/OpenCL/vendors/intel.icd" contain proper paths to the OpenCL driver
```
$ cat /etc/OpenCL/vendors/intel.icd 
/usr/lib/x86_64-linux-gnu/intel-opencl/libigdrcl.so
```
Note: path to the runtime lib may vary in different driver versions

## 6. Use LD_DEBUG=libs to trace loaded libraries
More details: https://github.com/bashbaug/OpenCLPapers/blob/markdown/OpenCLOnLinux.md







