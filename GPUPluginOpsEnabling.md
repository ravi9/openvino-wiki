# GPU plugin operations enabling flow

Common steps that are required to support a new operation in GPU plugin:
1. Read the [docs](https://github.com/openvinotoolkit/openvino/tree/master/docs/ops) for new operation 
2. Try to find existing primitive that fully or partially covers this operation.
3. Add new / extend existing cldnn primitive according to the operation spec.
4. Implement ref kernel that supports all parameters of the operation and all input/output data types and layouts
4. Add unit tests for the new operation
5. Add / update factory for this operation in the GPU plugin to use new primitive from earlier steps
6. Add functional single layer tests for the operation and try to cover most of the difference use cases of this operation
7. [Optional] If there are existing IRs with this operation, try to run the full model(s) to be sure that it's correctly processed within the context
8. Create PR with your changes
