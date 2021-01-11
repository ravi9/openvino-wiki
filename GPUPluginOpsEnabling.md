# GPU plugin operations enabling flow

Common steps that are required to support a new operation in GPU plugin:
1. Read the [docs](https://github.com/openvinotoolkit/openvino/tree/master/docs/ops) for new operation 
2. Try to find existing primitive that fully or partially covers this operation.
3. Add new / extend existing cldnn primitive according to the operation spec.
4. Implement reference **parallel** kernel that supports all parameters of the operation and all input/output data types and layouts
5. Add unit tests for the new operation
6. Add / update factory for this operation in the GPU plugin to use new primitive from earlier steps
7. Add functional single layer tests for the operation and try to cover most of the difference use cases of this operation
8. [Optional] If there are existing IRs with this operation, try to run the full model(s) to be sure that it's correctly processed within the context
9. [Optional] If there are existing IRs with this operation, try to run the full model(s) and estimate performance impact from this operation on total model execution time 
10. Create PR with your changes

