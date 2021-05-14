# CPU plugin debug capabilities

The page describes list of useful debug features, controlled by environment variables.
They can be activated at runtime and might be used for analyzing issues, getting more context, comparing execution results, etc.

To have CPU debug capabilities available at runtime the following CMake option should be used when building the plugin:
* `ENABLE_CPU_DEBUG_CAPS`. Default is `OFF`

The detailed information about each debug capability is available in repo's documentation:

[Debug capabilities](https://github.com/openvinotoolkit/openvino/blob/master/inference-engine/src/mkldnn_plugin/utils/README.md)