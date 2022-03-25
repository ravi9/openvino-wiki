# Build OpenVINO™ Inference Engine

## Contents

- [Introduction](#introduction)
- [Building for different OSes](#building-for-different-oses)
  - [[Windows* systems|BuildingForWindows]]
  - [[Linux* Systems|BuildingForLinux]]
  - [[macOS* Systems|BuildingForMacOS]]
  - [[Android* Systems|BuildingForAndroid]]
  - [[Raspbian Stretch* OS|BuildingForRaspbianStretchOS]]
- [Installing](#installing)
- [Adding OpenVINO Runtime (Inference Engine) to Your Project](#adding-openvino-runtime-inference-engine-to-your-project)
- [Building in a Docker image](#building-in-a-docker-image)
- [Use Custom OpenCV Builds](#use-custom-opencv-builds-for-inference-engine)
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
| [MULTI plugin](https://docs.openvinotoolkit.org/latest/openvino_docs_IE_DG_supported_plugins_MULTI.html) | Automatic inference on multiple devices simultaneously|
| [AUTO plugin](https://docs.openvinotoolkit.org/latest/openvino_docs_IE_DG_supported_plugins_AUTO.html) | Automatic device selection |

## Building for different OSes

  - [[Windows* systems|BuildingForWindows]]
  - [[Linux* Systems|BuildingForLinux]]
  - [[macOS* Systems|BuildingForMacOS]]
  - [[Android* Systems|BuildingForAndroid]]
  - [[Raspbian Stretch* OS|BuildingForRaspbianStretchOS]]

> **NOTE**: Please, refer to a [[dedicated guide|https://github.com/openvinotoolkit/openvino/wiki/CMakeOptionsForCustomCompilation]] with CMake options which control OpenVINO build if you need a custom build.

## Installing

Once project is built you can install OpenVINO™ Inference Engine into custom location:
 
```
cmake --install <BUILDDIR> --prefix <INSTALLDIR>
```

**Checking your installation:**

<details>
<summary>For versions prior to 2022.1</summary>
<p>

1. Obtaining Open Moldel Zoo tools and models

To have ability run samples and demo you need to clone Open Model Zoo repository and copy folder under `./deployment_tools` in your install directory:

```
git clone https://github.com/openvinotoolkit/open_model_zoo.git
cmake -E copy_directory ./open_model_zoo/ <INSTALLDIR>/deployment_tools/open_model_zoo/
```

2. Adding OpenCV to your environment

>You can find more info on OpenCV custom builds [here](#use-custom-opencv-builds-for-inference-engine).

Open Model Zoo samples use OpenCV functionality to load images, so to use it for demo builds you need to provide path to your OpenCV custom build by setting `OpenCV_DIR` environment variable and add path OpenCV libraries to `LD_LIBRARY_PATH (Linux*)` or `PATH (Windows*)` variable before running demos.

Linux:
```sh
export LD_LIBRARY_PATH=/path/to/opencv_install/lib/:$LD_LIBRARY_PATH
export OpenCV_DIR=/path/to/opencv_install/cmake
```

Windows:
```sh
set PATH=/path/to/opencv_install/bin/;%PATH%
set OpenCV_DIR=/path/to/opencv_install/cmake
```

3. Running demo

To check your installation go to the demo directory and run Classification Demo:

Linux and macOS:
```sh
cd <INSTALLDIR>/deployment_tools/demo
./demo_squeezenet_download_convert_run.sh
```

Windows:
```sh
cd <INSTALLDIR>\deployment_tools\demo
demo_squeezenet_download_convert_run.bat
```

Result:
```
Top 10 results:

Image <INSTALLDIR>/deployment_tools/demo/car.png

classid probability label
------- ----------- -----
817     0.6853030   sports car, sport car
479     0.1835197   car wheel
511     0.0917197   convertible
436     0.0200694   beach wagon, station wagon, wagon, estate car, beach waggon, station waggon, waggon
751     0.0069604   racer, race car, racing car
656     0.0044177   minivan
717     0.0024739   pickup, pickup truck
581     0.0017788   grille, radiator grille
468     0.0013083   cab, hack, taxi, taxicab
661     0.0007443   Model T

[ INFO ] Execution successful
```

</p>
</details>


<details open>
<summary> For 2022.1 and after</summary>
<p>

1. Build samples

To build the C++ sample applications, run the following commands:

Linux and macOS:
```sh
cd <INSTALLDIR>/samples/cpp
./build_samples.sh
```

Windows:
```sh
cd <INSTALLDIR>\samples\cpp
build_samples_msvc.bat
```

2. Install OpenVINO Development Tools

> **NOTE**: To build OpenVINO Development Tools (Model Optimizer, Post-Training Optimization Tool, Model Downloader and other Open Model Zoo tools) wheel package locally you are required to use CMake option: `-DENABLE_WHEEL=ON`.

To install OpenVINO Development Tools to work with Caffe models, execute the following commands:

Linux and macOS:

```sh
#setup virtual envrinment
python3 -m venv openvino_env
source openvino_env/bin/activate
pip install pip --upgrade

#install local package from install directory
pip install openvino_dev-<version>-py3-none-any.whl[caffe]  --find-links=<INSTALLDIR>/tools
```

Windows:
```bat
rem setup virtual envrinment
python -m venv openvino_env
openvino_env\Scripts\activate.bat
pip install pip --upgrade

rem install local package from install directory
cd <INSTALLDIR>\tools
pip install openvino_dev-<version>-py3-none-any.whl[caffe] --find-links=<INSTALLDIR>\tools
```

3.  Download the Models

Download the following model to run the Image Classification Sample:

Linux and macOS:
```sh
omz_downloader --name googlenet-v1 --output_dir ~/models
```

Windows:
```bat
omz_downloader --name googlenet-v1 --output_dir %USERPROFILE%\Documents\models
```

4. Convert the Model with Model Optimizer

Linux and macOS:
```sh
mkdir ~/ir
mo --input_model ~/models/public/googlenet-v1/googlenet-v1.caffemodel --data_type FP16 --output_dir ~/ir
```
Windows:
```bat
mkdir %USERPROFILE%\Documents\ir
mo --input_model %USERPROFILE%\Documents\models\public\googlenet-v1\googlenet-v1.caffemodel --data_type FP16 --output_dir %USERPROFILE%\Documents\ir
```

5. Run Inference on the Sample

Set up the OpenVINO environment variables:

Linux and macOS:
```sh
source <INSTALLDIR>/setupvars.sh
```

Windows:
```bat
<INSTALLDIR>\setupvars.bat
```

The following commands run the Image Classification Code Sample using the [`dog.bmp`](https://storage.openvinotoolkit.org/data/test_data/images/224x224/dog.bmp) file as an input image, the model in IR format from the `ir` directory, and on different hardware devices:

Linux and macOS:

```sh
cd ~/inference_engine_cpp_samples_build/intel64/Release
./classification_sample_async -i ~/Downloads/dog.bmp -m ~/ir/googlenet-v1.xml -d CPU
```

Windows:

```bat
cd  %USERPROFILE%\Documents\Intel\OpenVINO\inference_engine_samples_build\intel64\Release
.\classification_sample_async.exe -i %USERPROFILE%\Downloads\dog.bmp -m %USERPROFILE%\Documents\ir\googlenet-v1.xml -d CPU
```

When the sample application is complete, you see the label and confidence for the top 10 categories on the display:

```
Top 10 results:

Image dog.bmp

classid probability
------- -----------
156     0.6875963
215     0.0868125
218     0.0784114
212     0.0597296
217     0.0212105
219     0.0194193
247     0.0086272
157     0.0058511
216     0.0057589
154     0.0052615

```

</p>
</details>

## Adding OpenVINO Runtime (Inference Engine) to Your Project

<details>
<summary>For versions prior to 2022.1</summary>
<p>

For CMake projects, set the `InferenceEngine_DIR` and when you run CMake tool:

```sh
cmake -DInferenceEngine_DIR=/path/to/openvino/build/ .
```

Then you can find Inference Engine by [`find_package`]:

```cmake
find_package(InferenceEngine REQUIRED)
target_link_libraries(${PROJECT_NAME} PRIVATE ${InferenceEngine_LIBRARIES})
```
</p>
</details>


<details open>
<summary>For 2022.1 and after</summary>
<p>


For CMake projects, set the `OpenVINO_DIR` and when you run CMake tool:

```sh
cmake -DOpenVINO_DIR=<INSTALLDIR>/runtime/cmake .
```

Then you can find OpenVINO Runtime (Inference Engine) by [`find_package`]:

```cmake
find_package(OpenVINO REQUIRED)
add_executable(ov_app main.cpp)
target_link_libraries(ov_app PRIVATE openvino::runtime)

add_executable(ov_c_app main.c)
target_link_libraries(ov_c_app PRIVATE openvino::runtime::c)
```
</p>
</details>



## Use Custom OpenCV Builds

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

## Building in a Docker image
You can also build Intel® Distribution of OpenVINO™ toolkit in a Docker image by following this [guide](https://github.com/openvinotoolkit/docker_ci/tree/master/dockerfiles/ubuntu18/build_custom). 

## Next Steps

Congratulations, you have built the OpenVINO Runtime (Inference Engine). To get started with the
OpenVINO™, proceed to the Get Started guides:

* [Get Started with the OpenVINO™ on Linux*](GettingStarted)

## Additional Resources

* [OpenVINO™ Release Notes](https://software.intel.com/en-us/articles/OpenVINO-RelNotes)
* [Introduction to Intel® OpenVINO™ toolkit](https://docs.openvinotoolkit.org/latest/_docs_IE_DG_Introduction.html)
* [Inference Engine Samples Overview](https://docs.openvinotoolkit.org/latest/_docs_IE_DG_Samples_Overview.html)
* [Inference Engine Developer Guide](https://docs.openvinotoolkit.org/latest/_docs_IE_DG_Deep_Learning_Inference_Engine_DevGuide.html)
* [Model Optimizer Developer Guide](https://docs.openvinotoolkit.org/latest/_docs_MO_DG_Deep_Learning_Model_Optimizer_DevGuide.html)

---
\* Other names and brands may be claimed as the property of others.


[https://download.01.org/opencv/2020/openvinotoolkit]:https://download.01.org/opencv/2020/openvinotoolkit
[build instructions]:https://docs.opencv.org/master/df/d65/tutorial_table_of_content_introduction.html
[`find_package`]:https://cmake.org/cmake/help/latest/command/find_package.html