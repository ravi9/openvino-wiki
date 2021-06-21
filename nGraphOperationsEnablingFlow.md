1. Implement operation "shell" in the `ngraph/core/[src|include]/op/`:
* Implement constructor(s)
* Implement `validate_and_infer_types` method which should support dynamic input tensor(s) (with partially dynamic shapes)
* Implement `visit_attributes` method
* Implement `clone_with_new_inputs` method. The generated operation version must be explicitly specified and be equal to the operation version being added
* In `*.hpp` file add: `NGRAPH_RTTI_DECLARATION;`
* In `*.cpp` file add:
```cpp
NGRAPH_RTTI_DEFINITION(op::vX::<Operation>, "<Operation>", X);
```
* To support conditional compilation add following to `*.cpp` file :
```cpp
NGRAPH_OP_SCOPE(<operation_version>_<operation_name>_<method_name>);
```
* Add shape infer unit-tests to the `ngraph/test/type_prop/`

2. Add operation to the dedicated opset file `ngraph/core/include/ngraph/opsets/opsetX_tbl.hpp`

3. Implement `evaluate` method for the operation (reference implementation) in the `ngraph/core/reference/[src|include]/ngraph/runtime/reference/`. Reference implementation can be called from interpreter backend (file `ngraph/test/runtime/interpreter/evaluates_map.cpp`) or from ngraph core (`ngraph/core/src/op/your_operation_name.cpp/hpp`). To not increase the binary size of ngraph lib it should be placed in interpreter backend unless you are directly asked to put it in the ngraph core. While adding reference implementation the following points should be considered:
* The method should avoid using the template parameters if possible. However, for the small operations like activation functions it is acceptable.
* The method should be instantiated for practical data types only.
* If the method can be implemented without defining strict data types (for example, data movement operations like Concat or Split) then it should be implemented in a type-agnostic way. 