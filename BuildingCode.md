# Build OpenVINO™ Inference Engine

## Contents

- [Introduction](#introduction)
- [Building for different OSes](#building-for-different-oses)
  - [[Windows* systems|BuildingForWindows]]
  - [[Linux* Systems|BuildingForLinux]]
  - [[macOS* Systems|BuildingForMacOS]]
  - [[Android* Systems|BuildingForAndroid]]
  - [[Raspbian Stretch* OS|BuildingForRaspbianStretchOS]]
- [Use Custom OpenCV Builds for Inference Engine](#use-custom-opencv-builds-for-inference-engine)
- [Add Inference Engine to Your Project](#add-inference-engine-to-your-project)
- [Next Steps](#next-steps)
- [Additional Resources](#additional-resources)

## Introduction

The Inference Engine can infer models in different formats with various input
and output formats.

The open source version of Inference Engine includes the following plugins:

| PLUGIN               | DEVICE TYPES |
| ---------------------| -------------|
| CPU plugin           | Intel® Xeon® with Intel® AVX2 and AVX512, Intel® Core™ Processors with Intel® AVX2, Intel® Atom® Processors with Intel® SSE |
| GPU plugin           | Intel® Processor Graphics, including Intel® HD Graphics and Intel® Iris® Graphics |
| GNA plugin           | Intel® Speech Enabling Developer Kit, Amazon Alexa\* Premium Far-Field Developer Kit, Intel® Pentium® Silver processor J5005, Intel® Celeron® processor J4005, Intel® Core™ i3-8121U processor |
| MYRIAD plugin        | Intel® Neural Compute Stick 2 powered by the Intel® Movidius™ Myriad™ X |
| Heterogeneous plugin | Heterogeneous plugin enables computing for inference on one network on several Intel® devices. |

## Building for different OSes

  - [[Windows* systems|BuildingForWindows]]
  - [[Linux* Systems|BuildingForLinux]]
  - [[macOS* Systems|BuildingForMacOS]]
  - [[Android* Systems|BuildingForAndroid]]
  - [[Raspbian Stretch* OS|BuildingForRaspbianStretchOS]]

## Use Custom OpenCV Builds for Inference Engine

> **NOTE**: The recommended and tested version of OpenCV is 4.4.0.

Required versions of OpenCV packages are downloaded automatically during the
building Inference Engine library. If the build script can not find and download
the OpenCV package that is supported on your platform, you can use one of the
following options:

* Download the most suitable version from the list of available pre-build
  packages from [https://download.01.org/opencv/2020/openvinotoolkit] from the
  `<release_version>/inference_engine` directory.

* Use a system-provided OpenCV package (e.g with running the
  `apt install libopencv-dev` command). The following modules must be enabled:
  `imgcodecs`, `videoio`, `highgui`.

* Get the OpenCV package using a package manager: pip, conda, conan etc. The
  package must have the development components included (header files and CMake
  scripts).

* Build OpenCV from source using the [build instructions](https://docs.opencv.org/master/df/d65/tutorial_table_of_content_introduction.html) on the OpenCV site.

After you got the built OpenCV library, perform the following preparation steps
before running the Inference Engine build:

1. Set the `OpenCV_DIR` environment variable to the directory where the
   `OpenCVConfig.cmake` file of you custom OpenCV build is located.
2. Disable the package automatic downloading with using the `-DENABLE_OPENCV=OFF`
   option for CMake-based build script for Inference Engine.

## Add Inference Engine to Your Project

For CMake projects, set the `InferenceEngine_DIR` environment variable:

```sh
export InferenceEngine_DIR=/path/to/openvino/build/
```

Then you can find Inference Engine by `find_package`:

```cmake
find_package(InferenceEngine REQUIRED)
target_link_libraries(${PROJECT_NAME} PRIVATE ${InferenceEngine_LIBRARIES})
```

## Next Steps

Congratulations, you have built the Inference Engine. To get started with the
OpenVINO™, proceed to the Get Started guides:

* [Get Started with Deep Learning Deployment Toolkit on Linux*](get-started-linux.md)

## Additional Resources

* [OpenVINO™ Release Notes](https://software.intel.com/en-us/articles/OpenVINO-RelNotes)
* [Introduction to Intel® Deep Learning Deployment Toolkit](https://docs.openvinotoolkit.org/latest/_docs_IE_DG_Introduction.html)
* [Inference Engine Samples Overview](https://docs.openvinotoolkit.org/latest/_docs_IE_DG_Samples_Overview.html)
* [Inference Engine Developer Guide](https://docs.openvinotoolkit.org/latest/_docs_IE_DG_Deep_Learning_Inference_Engine_DevGuide.html)
* [Model Optimizer Developer Guide](https://docs.openvinotoolkit.org/latest/_docs_MO_DG_Deep_Learning_Model_Optimizer_DevGuide.html)

---
\* Other names and brands may be claimed as the property of others.


[https://download.01.org/opencv/2020/openvinotoolkit]:https://download.01.org/opencv/2020/openvinotoolkit
[build instructions]:https://docs.opencv.org/master/df/d65/tutorial_table_of_content_introduction.html
