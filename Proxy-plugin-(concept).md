# Motivation
 - OpenVINO may have multiple hardware plugins for similar device type from different vendors (e.g. Intel and NVidia GPUs) and currently user must address them with different names ("GPU" and "NVIDIA" respectively). Using same name for such cases seems to be more user-friendly approach.
 - Moreover, single physical device may have multiple plugin which support it. For example, Intel GPU plugin is OpenCL based, thus can run on other OCL-compatible devices including NVIDIA gpus. In that case we may have primary plugin ("NVIDIA") which provides best performance on target device and fallback plugin ("INTEL_GPU") which helps to improve models coverage for the cases when primary plugin has limited operation set support, and both plugins may be executed via HETERO mode.
- Implicit HETERO plugin usage may be extended to run on different device types - HETERO:xPU,CPU

# Proposal
- Introduce proxy plugin with the following responsibilities:
    - Providing user-visible aliases which aggregates lower-level plugins ("GPU" alias for "INTEL_GPU" and "NVIDIA_GPU")
    - Hide real hardware plugin names under the common device name
- Proxy plugin should provide the optimal performance with minimum overhead
- Implicit HETERO mode run for the cases when multiple plugins can be used for the same device
- Managing properties to configure target and fallback devices.

![image](https://user-images.githubusercontent.com/4737252/188442276-bf51590c-2dcd-430b-bc6e-17630306b391.png)

# Requirements

- Do not provide additional libraries and don't affect load time (proxy plugin is a part of `openvino` library)
- No overhead for load inference time
    - Fallback to hardware plugin if device is supported only by one plugin
    - Minimal overhead in case of multiple plugins for one device
        - Plus one call of query network if entire model can be executed on preferable plugin
        - Hetero execution in other case
- Allow to configure device

# Behavior details

- `ov::Core` can create several instances of proxy plugins (separate instance for each high-level device)
- Configure from `plugins.xml`

![image](https://user-images.githubusercontent.com/4737252/188448082-a0321c7c-18fc-4708-8e52-f43948508298.png)

- Proxy plugin uses `ov::device::uuid` property to match devices from different plugins
- Plugin allows to set properties of primary plugin, in case of configuration fallback plugin, user should use `ov::device::properties()`
- Plugin doesn't use the system of devices enumeration of hidden plugins and use ids for enumeration (`DEV.0`, `DEV.1`, ..., `DEV.N` and etc.)
