# Propagating shapes for nGraph Function

`nGraph::Function` represents model with operations and their relations describing how data would flow through the function during the inference.
Data size estimations for output tensors of each operation is required for efficient model execution.
Such an estimation depends on the data type and shape of intermediate tensors of the model.
For publicly available operations you can see type and shape propagation rules in the !!!opset documentation!!!  

Shape propagation is a process of output shape calculation for each operation in the graph.
It starts from all the `Parameter` operations in the `ngraph::Function` and continues to calculate output shapes for all the operations in the `ngraph::Function` in the topological order.
Method `Op::validate_and_infer_types` is responsible for operation validation, shape and type propagation.  

## nGraph shapes representation

Shapes could be represented as `ngraph::Shape` or as `ngraph::PartialShape` in nGraph.
`ngraph::Shape` could be used in case rank and all the shape dimensions are static. 
`ngraph::PartialShape` could be used for both static and dynamic shape representations.

Depending on the amount of known information about the rank and shape dimensions we can formulate shapes in different ways.
Here are some examples of shape creation:
<pre>
1. Shape with undefined rank:
    ngraph::PartialShape::dynamic();
2. Shape with defined rank and fully undefined dimensions:
    ngraph::PartialShape({1, 3, Dimension::dynamic(), Dimension::dynamic()});  // rank == 4, two static dimensions and two fully dynamic dimension
    ngraph::PartialShape({Dimension::dynamic(), Dimension::dynamic()});  // rank == 2, all dimensions are fully dynamic
    ngraph::PartialShape::dynamic(5); // rank == 5, all dimensions are fully dynamic
3. Shape with defined rank and undefined dimensions with specified range:
    ngraph::PartialShape({{1, 8}, 3, 400, 400}); // rank == 4, first dimension is dynamic and can be any number from 1 to 8 inclusive, all the other dimensions are static
    ngraph::PartialShape({{2, -1}, 3, 400, 400}); // rank == 4, first dimension is dynamic and can be any number larger or equal 2, all the other dimensions are static
    ngraph::PartialShape({{-1, 8}, 3, 400, 400}); // rank == 4, first dimension is dynamic and will not be larger than 8, all the other dimensions are static
4. Shape with defined rank and fully defined dimensions:
    ngraph::PartialShape({1, 3, 400, 400});  // rank == 4
    ngraph::Shape({1, 3, 400, 400});  // rank == 4
    ngraph::PartialShape({5});  // rank == 1, one-dimensional tensor with five values in it
    ngraph::PartialShape({});  // rank == 0, scalar -- zero-dimensional tensor with single value in it
    ngraph::PartialShape({1});  // rank == 1, one-dimensional tensor with single value in it
    ngraph::PartialShape({1, 3, 0, 0});  // rank == 4, four-dimensional tensor with no value in it empty tensor
</pre>

`ngraph::PartialShape` stores `ngraph::Rank` for the shape rank. It could be fully undefined or static.
`ngraph::PartialShape` stores shape values as a vector of `ngraph::Dimension` for the case of static `ngraph::Rank`. 

`ngraph::Dimension` is represented by an interval which is a pair of values -- maximum and minimum value for the dimension, for both static and dynamic cases. 
They are equal for static dimensions and are set to different values for range and fully dynamic dimensions.

<pre>
1. dynamic ngraph::Dimension:
    ngraph::Dimension::dynamic();
    ngraph::Dimension(-1);  // special value for Dimension
    ngraph::Dimension{0, MAX_INT};
2. interval ngraph::Dimension:
    ngraph::Dimension{1, 10}; 
    ngraph::Dimension{10, 1}; // range values would be flipped, such a dimension is equal to previous example 
    ngraph::Dimension{0, MAX_INT};
3. static ngraph::Dimension
    ngraph::Dimension{3};
    ngraph::Dimension{3, 3};
</pre>

We allow creation of negative dimension on `ngraph::Interval`, `ngraph::Dimension`, `ngraph::PartialShape` level due to historical reasons, but there is no point in it in terms of shape propagation.
> NOTE: Dimension(-1) is equal to Dimension::Dynamic(). There is no way to create static Dimension with value -1

## Shape propagation

The goal of shape propagation is to keep and pull all the known information about output shapes dimensions from the operation semantics.
During the shape propagation we may need input ranks/shapes for the operation or maybe even input values.

Examples of shape propagation:

[`SoftMax` example](../ops/activation/ReLU_1.md):
Semantics of *SoftMax* operation states that operation leaves shape of incoming data unchanged.
From the shape propagation standpoint we should set output shape equal to input shape without any checks.

> Additionally *SoftMax* operation has `axis` attribute which represents the axis of which the *SoftMax* is calculated.
`axis` value should be within `0..rank(input_data)-1` range to be compliant with input data rank.
As input data rank can be dynamic `validate_and_infer_types` method would contain conditional behaviour depending on the input rank. 
Those construction should not affect output shape setting, as they are only connected with validation.

Examples:
<pre>
? == Dimension::dynamic()

Static input shape:
    axis = 1
    input  shape: [1, 1000]
    output shape: [1, 1000]

Static input shape, wrong axis:
    axis = 3
    input shape: [1, 1000]
    Validation failed due to rank is out of `0..rank(input_data)-1` range

Dynamic input shape, static input rank:
    axis = 1
    input  shape: [{1, 8}, ?, ?, ?]
    output shape: [{1, 8}, ?, ?, ?]

Dynamic input shape, static input rank, wrong input rank:
    axis = 7
    input shape: [?, ?, ?, ?]
    Validation failed due to rank is out of `0..rank(input_data)-1` range

Dynamic input shape, dynamic input rank:
    axis = 10
    input  shape: PartialShape::dynamic()
    output shape: PartialShape::dynamic()
    Validation passed as rank is unknown at the moment of validation and we assume that it is valid in case we are not able to validate it yet.
</pre>


[`Concat` example](../ops/movement/Concat_1.md):
*Concat* operation semantics requires:
- input shapes to have the same rank
- input shape dimensions should be equal among all the inputs except the `axis` dimension index
These are input shape restrictions to validate in `opset1::Concat::validate_and_infer_types`.

Output shape should be the same as one of input shape but with `axis` dimension equal to sum of all the axis dimensions among input shapes.

Examples:
<pre>
Static input shapes:
    axis = 1
    1st input shape: [1, 2, 3, 4]
    2nd input shape: [1, 5, 3, 4]    
    output shape:    [1, 7, 3, 4]

Dynamic axis dimension of input shape:
    axis = 1
    1st input shape: [1,        2, 3, 4]
    2nd input shape: [1, {10, 15}, 3, 4]    
    output shape:    [1, {12, 17}, 3, 4]

Dynamic non axis dimension of input shape:
    axis = 1
    1st input shape: [1, 2, 3, {1, 5}]
    2nd input shape: [1, 5, 3, 4]
    output shape:    [1, 7, 3, 4]
    
    Validation passes as 4th dimension range of 1st input shape intersects with 4th dimension range of 2nd input shape. 
    Meaning that from all the possible variants of 1st input shape there is a valid one.
    4th dimension of output shape is set to the Output 

Dynamic non axis dimension of input shape:
    axis = 1
    1st input shape: [1, 2, 3, {1, 5}]
    2nd input shape: [1, 5, 3, 4]
    output shape:    [1, 7, 3, 4]
    
    Validation passes as 4th dimension range of 1st input shape intersects with 4th dimension range of 2nd input shape. 
    Meaning that from all the possible variants of 1st input shape there is a valid one.
    4th dimension of output shape is set to the Output 

Dynamic input rank:
    axis = -3
    1st input shape: [1, 2, 3, {1, 5}]
    2nd input shape: PartialShape::dynamic()
    output shape:    [1, {2, MAX_INT}, 3, {1, 5}]
    
    Axis normalization is applicable as there is at least one input with static rank and we assume that all the other inputs would be compliant with operation semantics.
    `axis` dimension would be a sum of all the input `axis` dimensions. 
    As `axis` dimension of 1st input is equal to 2, sum of input `axis` dimensions wouldn't .Сbe less than 2.
    That is why we set minimum of `axis` dimension of output shape to 2.
    Maximum of `axis` dimension of output shape is unknown so we set it to dynamic.
    All the output dimensions except the `axis` one are set to the corresponding input dimensions.

Dynamic input shapes, static ranks:
    axis = -3
    1st input shape: [     1,      2,      3, {1, 5}]
    2nd input shape: [{1, 5}, {1, 5}, {1, 5}]     
    
    Validation fail -- rank of both inputs are static and they are different.

Dynamic input shapes:
    axis = 1
    1st input shape: [{0, 1}, {2, 3}, {4, 7}]
    2nd input shape: [{1, 2}, {3, 4}, {5, 10}]     
    output shape:    [     1, {5, 7}, {5, 7}]
    
    Validation passes -- all the requirements satisfied.
    To determine first dimension of output shape  Intersection of first dimensions of input  7
</pre>

Shape propagation may require not only input shapes but actual values that operation consumes.
Usually (not always) those are inputs delivering axes, indices, etc. 
These inputs can be represented not as Constant operation but as a sub-graph.
To easily access the value of incoming tensor of operation during the shape propagation we have to trigger value propagation for particular tensor. 

