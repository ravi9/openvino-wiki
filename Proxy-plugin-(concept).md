# Motivation
 - OpenVINO may have multiple hardware plugins for similar device type from different vendors (e.g. Intel and NVidia GPUs) and currently user must address them with different names ("GPU" and "NVIDIA" respectively). Using same name for such cases seems to be more user-friendly approach.
 - Moreover, single physical device may have multiple plugin which support it. For example, Intel GPU plugin is OpenCL based, thus can run on other OCL-compatible devices including NVIDIA gpus. In that case we may have primary plugin ("NVIDIA") which provides best performance on target device and fallback plugin ("INTEL_GPU") which helps to improve models coverage for the cases when primary plugin has limited operation set support, and both plugins may be executed via HETERO mode.
- Implicit HETERO plugin usage may be extended to run on different device types - HETERO:xPU,CPU

# Proposal
- Introduce proxy plugin with the following responsibilities:
 - Providing user-visible aliases which aggregates lower-level plugins ("GPU" alias for "INTEL_GPU" and "NVIDIA_GPU")
- Implicit HETERO mode run for the cases when multiple plugins can be used for the same device
- Managing properties to configure target and fallback devices.

picture (TBD)

# Requirements

TBD

# Behavior details

TBD
