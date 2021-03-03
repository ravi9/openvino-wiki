# OpenVINO package footprint minimization

## Contents

* [Introduction](#introduction)
* [Required components](#required-components)
* [IE Readers](#ie-readers)
* [IE Plugins](#ie-plugins)
* [Other opportunities](#other-opportunities)

## Introduction

This guide provides you with the information that will help you to decrease the OpenVINO package binary footprint. With this guide, you will learn approaches to reduce OpenVINO binary size and understand which OpenVINO components are required for your pipeline in order to deploy only necessary components. 

This guide is divided to several sections with required and optional components.

## Required components

* **ngraph** is required for all OpenVINO tools to represent Neural Network inside the OpenVINO components.
* **inference_engine** is required for all runtime components.
* **inference_engine_legacy*** is required for majority Inference Engine plugins which use the legacy API (we have a plan to remove this component after migration from legacy API).
* **inference_engine_transformations** is required for all plugins, this library contains a lot of transformations for network optimizations and modifications.
* **inference_engine_lp_transformations** is required for all IE plugins which support low precision networks.

## IE Readers

* **inference_engine_ir_reader** is required only to read IR files. If you don't read models from IR you can skip this component.
* **inference_engine_onnx_reader** is required only to read ONNX models. If you don't read ONNX models you can skip this component.
    * **onnx_importer** is required for ONNX reader, you can skip this component if you don't use ONNX reader.

## IE Plugins

* **clDNNPlugin** is required for inference on GPU device.
* **GNAPlugin** is required for inference on GNA device.
* **HeteroPlugin** is required for heterogeneous inference. It is needed if we have primitives which are not supported on some devices.
* **MKLDNNPlugin** is required for inference on CPU device.
* **MultiDevicePlugin** is required for parallel inference on several devices.
* **myriadPlugin** is required for inference on VPU device.

Also you can use CMake parameters to disable plugins compilation.

## Other opportunities

* [[Conditional compilation|ConditionalCompilation]] allows to build custom OpenVINO tools to support limited set of models.
* **tbb**, **omp** libraries are required for parallel execution, by default OpenVINO uses **TBB**, but to reduce binary size you can try to use **OpenMP**, or disable parallel execution (`-DTHREADING=SEQ`) to avoid dependencies to **TBB** and **OpenMP**.