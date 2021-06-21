Implement operation "shell" in the `ngraph/core/[src|include]/op/`:
* Implement constructor(s)
* Implement `validate_and_infer_types` method which should support dynamic input tensor(s) (with partially dynamic shapes)
* Implement `visit_attributes` method
* Implement `clone_with_new_inputs` method. The generated operation version must be explicitly specified and be equal to the operation version being added
* In hpp file add: `NGRAPH_RTTI_DECLARATION;`
* In cpp file add: `NGRAPH_RTTI_DEFINITION(op::vX::<Operation>, "<Operation>", X);`
* In cpp file add: `NGRAPH_OP_SCOPE(<operation_version>_<operation_name>_<method_name>);`  in order to support conditional compilation
* Add shape infer unit-tests in the `ngraph/test/type_prop/`