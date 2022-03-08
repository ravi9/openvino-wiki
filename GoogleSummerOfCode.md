# Google Summer of Code

Spend your summer doing something interesting and valuable for the open-source community and join Google Summer of Code. Read more information about how the program works on [this page](https://summerofcode.withgoogle.com/).

OpenVINO Toolkit is applying as a mentoring organization for 2022.

## Prerequisite task

We require one new notebook pull request sent to our OpenVINO notebooks repository from each potential GSoC contributor before accepting participation for GSoC. In addition, we would like to see if you know how to code and your coding style. To fulfill this requirement please:
1. Fork [OpenVINO Notebooks](https://github.com/openvinotoolkit/openvino_notebooks) repository.
2. Implement a new notebook as a [2xx notebook](https://github.com/openvinotoolkit/openvino_notebooks#model-demos) to demonstrate one of your favorite models from Open Model Zoo ([Intel](https://github.com/openvinotoolkit/open_model_zoo/blob/master/models/intel/index.md) or [public](https://github.com/openvinotoolkit/open_model_zoo/blob/master/models/public/index.md) one) (please don't implement notebooks for models with demos already done or in progress - you can find them [here](https://github.com/openvinotoolkit/openvino_notebooks#model-demos) and [here](https://github.com/openvinotoolkit/openvino_notebooks/pulls)).
3. The notebook should be consistent with others in the same branch and should fit [the requirements](https://github.com/openvinotoolkit/openvino_notebooks/blob/main/CONTRIBUTING.md) for the contribution.
4. Your notebook should have descriptions and comments proving you know what you do (similar to descriptions and comments in e.g., [403 notebook](https://github.com/openvinotoolkit/openvino_notebooks/blob/main/notebooks/403-action-recognition-webcam/403-action-recognition-webcam.ipynb)).
5. Create a pull request.

## Application template

Your application should consist of the following parts:
1. About you
    1. Your full name
    2. Your university/current enrollment
    3. Short bio
    4. Your experience in programming (especially C++ and Python)
    5. Your experience in ML and DL
2. About the project
    1. What is your choice?
    2. Why did you choose this specific idea?
    3. How much time do you plan to invest in the project? 
    4. Provide an abstract of the solution
    5. Provide a detailed timeline of how you want to implement the project (include the main points you want to cover and dates)
3. Answer the questions:
    1. How do you know OpenVINO?
    2. What do you know about OpenVINO? 
    3. How could you apply it to your professional development?
    4. Describe any other career development plan you have for the summer in addition to GSoC.
    5. Why should we pick you?
4. Tasks
    1. Screenshot of **running** object detection ([401-object-detection-webcam notebook](https://github.com/openvinotoolkit/openvino_notebooks/blob/main/notebooks/401-object-detection-webcam/401-object-detection.ipynb)) – please take a screenshot for the last code cell
    2. Link to your pull request (for the prerequisite task – the top part of this document)

Proposal examples can be found [here](https://google.github.io/gsocguides/student/proposal-example-1) and [here](https://google.github.io/gsocguides/student/proposal-example-2). Please contact early to discuss your application with the mentor.

## Project ideas for 2022

### 1. Object Detection demo application to run AI on ARM-based Android devices

**Short description:**
ARM CPUs support has been added to Inference Engine via the dedicated [ARM CPU plugin](https://github.com/openvinotoolkit/openvino_contrib/tree/master/modules/arm_plugin). ARM processors are widely used in Android smartphones, so we want to develop an Android demo application that demonstrates plugin possibilities on this platform. The demo should be written in Java and use Java wrappers to reach Inference Engine public [API](https://github.com/openvinotoolkit/openvino_contrib/tree/master/modules/java_api). We suggest reviewing the functionality of OMZ [object detection demo](https://github.com/openvinotoolkit/open_model_zoo/tree/master/demos/object_detection_demo/cpp) and propagating its core functionality to the Android demo. 

**Expected outcomes:**
Android demo application with object detection functionality. Any model (or several models) from the [list](https://github.com/openvinotoolkit/open_model_zoo/blob/master/demos/object_detection_demo/cpp/models.lst) supported by the plugin could be used. 

**Skills required/preferred:**
Practical experience with developing Java apps for Android, at least a basic understanding of computer vision and deep learning technologies, enough to run a network and make it run at real-time speed

**Mentors:**
Alexander Nesterov

**Size of project:**
350 hours

**Difficulty:**
Medium

### 2. OpenVINO inference engine WASM build

**Short description:**
OpenVINO Inference Engine is a set of C++ libraries with C and Python bindings providing a common API to deliver high performance deep learning inference solutions on multi-platform including Windows, Linux, and Mac OS. We would like to extend these platforms by providing WASM build for the inference engine to enable the web/JS developers to use OpenVINO inference engine in the web browser.

**Expected outcomes:**
* WASM build of OpenVINO inference engine
* JavaScript and TypeScript friendly API for running inference in the browser using OpenVINO WASM build
* Sample Projects and documentation

**Skills required/preferred:**
C++, JS, WASM and Emscripten

**Mentors:**
Radwan Ibrahim

**Size of project:**
350 hours

**Difficulty:**
Medium to hard

### 3. Add support of Apple M1 to OpenVINO using BNNS library

**Short description:**
OpenVINO is an open-source product consisting of a set of tools and C++ runtime libraries which allows to perform inference on different hardware including Intel accelerators such as CPU, GPU, GNA, MYRIAD, ARM CPU, NVIDIA GPU, in a unified manner - write an application once using OpenVINO Runtime API and deploy on different OSes/hardware. The goal of this GSOC project is to introduce native support for Apple platforms (M1 and forward) by leveraging AI accelerators via the BNNS library-based plugin for OpenVINO. The work supposes the implementation of the plugin with all details: identify matching between OpenVINO operations and BNNS layers, adding graph transformations to convert between them, creation of BNNS graph, construction of optimal inference pipeline to execute the BNNS graph of operations, and take care of memory management.

**Expected outcomes:**
Minimal working OpenVINO plugin which is able to perform inference via BNNS backend to offload heavy operations to Apple M1 accelerator, the plugin should demonstrate performance comparable to direct usage of Apple software, provide accurate results, have scalable architecture.  

**Skills required/preferred:**
C++, objective-c/swift, the interaction of C++ and objective-c/swift, system programming, deep learning frameworks, graph optimizations, multi-threading programming, gtest, cross-platform make

**Mentors:**
Ilya Lavrenov

**Size of project:**
350 hours

**Difficulty:**
Medium to hard

### 4. Add OpenVINO support to Google Mediapipe

**Short description:**
Mediapipe is a customizable platform for ML solutions for media processing. Intel OpenVINO is a set of C++ libraries with C and Python bindings providing a common API to deliver high-performance deep learning inference solutions. MediaPipe currently supports CNN inference using TensorFlow and TFLite frameworks. We want to add support for OpenVINO inference to MediaPipe. To do it, we need to develop new calculator classes supporting OpenVINO.
Suggested calculator types:

* Minimal task (detects object according to provided model):
  * Object detection calculator:
    * Input: IMAGE:image
    * Ouput: DETECTION:detections
* Maximal task
  * BERT question answering calculator:
    * Input: TENSOR: tokens
    * Output: Start-end indexes
  * Tokenizer calculator:
    * Input: Text or URL (may be split it into 2 different calculators)
    * Ouput: TENSOR: tokens
  * Index_to_Text calculator:
    * Input: Text or URL
    * Input: start-end indexes
    * Ouput: text

Inference calculators should be created using the ModelAPI framework (part of OpenVINO Model Zoo). Also, examples with a full pipeline should be created.

**Expected outcomes:**
Working examples for Object detection and BERT question answering working with OpenVINO-based calculators.

**Skills required/preferred:**
C++ knowledge, basic knowledge of OpenVINO framework, experience with Google MediaPipe is a plus

**Mentors:**
Fedor Zharinov, Vladimir Zlobin

**Size of project:**
350 hours

**Difficulty:**
Medium to hard (depending on chosen scope)

### 5. 3D Jupyter Notebooks and WebGL Integration for OpenVINO

**Short description:**
The world is 3D but in AI development we often work on flat 2D displays with flat data visualization plots and charts. In machine learning, many of the tasks may be much better understood if we provide a 3D or 4D (space and time) perspective. In this project, we will answer this question by providing the beginner-friendly Jupyter 3D engine for machine learning visualization along with WebGL integration. Our main goal is not only to make 3D or 4D datasets easier to visualize, but also to make machine learning easier to understand in a more humanistic way.

**Expected outcomes:**
Working 3D support for Jupyter Notebooks running OpenVINO (AI inference). For example, visualizing 3D body pose and characters, visualization 2D-3D mapping, and also providing a clean interface to set up these without a 3D engine or graphics programming background.

**Skills required/preferred:**
Understanding of graphics pipelines and (GPU programming is a plus), software Engineer background and code releasing, Python, C++

**Mentors:**
Raymond Lo

**Size of project:**
350 hours

**Difficulty:**
Medium to hard

### 6. Train a DL model for synthetic data generation for model optimization

**Short description:**
In most cases, DL model optimization requires the presence of real data that the user should provide for the optimization method (e.g. quantization or pruning). However, it was noticed that such methods can perform quite well on synthetic data which are not even relevant to the use case. In this task, we will create a synthetic GAN or VAE based DL model that is capable of generating synthetic images based on the text hints provided by the user. This data will be used to evaluate the 8-bit post-training quantization method on a wide range of Computer Vision models.

**Expected outcomes:**
DL model that generates synthetic data based on the text input.

**Skills required/preferred:**
DL model training, understanding of GAN, VAE architectures

**Mentors:**
Alexander Kozlov, Lyubov Talamanova

**Size of project:**
350 hours

**Difficulty:**
Medium

### 7. Showcase performance of PyTorch Image Models (timm) with OpenVINO

**Short description:**
DL model inference performance is a hop trend of recent years that is aimed at bringing AI into real-world applications. And OpenVINO is a famous DL inference solution that delivers best-in-class performance on Intel Architectures. In this task, we will provide a Jupyter notebook/tutorial (similar to [this one](https://github.com/rwightman/pytorch-image-models/blob/master/notebooks/EffResNetComparison.ipynb)) where we showcase inference performance on the set of popular DL models that come from the PyTorch Image Models project on GitHub (timm). The tutorial will provide details on how to install OpenVINO, convert models to OpenVINO Intermediate Representation, and do the proper benchmarking under the various settings.

**Expected outcomes:**
Pull-request with the tutorial into PyTorch Image Models.

**Skills required/preferred:**
DL basics, understanding of ML model optimization 

**Mentors:**
Alexander Kozlov

**Size of project:**
175 hours

**Difficulty:**
Easy

### 8. NEON optimized Snippets Emitters for OpenVINO Arm Plugin

**Short description:**
Snippets is a new optimization technique introduced in OpenVINO to speed up memory access bound operations. Snippets provides the capability to fuse several consequent DNN layers processing into a single operation that could be processed in a memory access efficient way using JIT generation of processing code. Code generation consists of a few stages, however, only generation of optimized code for specific operations are platform-specific and cannot be easily ported from existing implementation for x86 architecture. Therefore the development of platform-specific Emitters that implement code generation API will make possible further improvement of OpenVINO framework performance on the Arm platform.

**Expected outcomes:**
Implementation of Emitter classes for operations from opset1

**Skills required/preferred:**
C++, understanding of DNN model structure, understanding of JIT code generation concept, performance optimization experience, knowledge of NEON instruction set 

**Mentors:**
Vitaly Tuzov, Alexander Nesterov

**Size of project:**
350 hours

**Difficulty:**
Medium to hard

## Contact us

[Mailing list](https://lists.01.org/postorius/lists/gsoc.lists.01.org/)