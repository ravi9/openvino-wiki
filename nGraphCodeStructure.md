# nGraph code structure

Historycally nGraph was a standalone library for graph representation, but at some point nGraph became a part of OpenVINO and now nGraph is a part of [OpenVINO repository](https://github.com/openvinotoolkit/openvino/tree/master/ngraph).

Let's focus on the structure of nGraph source code:
```
- ngraph/ - root nGraph folder
    - cmake/ - nGraph specific CMake scripts
    - core/ - nGraph core library
        - builder/ - static library with builders for some nGraph operations
        - include/ - nGraph library public API
        - reference/ - static library with nGraph reference implementations
        - src/ - sources of nGraph library
    - frontend/ - nGraph frontends
        - onnx_common/ - static library with common ONNX utilities
        - onnx_editor/ - static library with the ONNX Editor, the component allows to modify original ONNX model
        - onnx_import/ - ONNX importer library, importer allows to translate ONNX model to nGraph representation
    - python/ - nGraph Python API
        - src/ - spirces of nGraph Python API
        - tests/ - tests for Pyton API
    - test/ - nGraph unit tests
        - backend/ - nGraph backends tests, tests runs on Inference Engine CPU, GPU plugins and reference implementations
        - conditional_compilation/ - tests cover conditional compilation feature
        - files/ - test data
        - models/ - test models
        - onnx/ - ONNX tests
        - op_eval/ test cover evaluate methods
        - runtime/ - nGraph backends
            - ie/ - Inference Engine backend
            - interpreter/ - Interpreter backend
        - type_prop/ - type and shape propagation tests
        - util/ - nGraph test utility library
```
