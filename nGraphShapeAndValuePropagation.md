# Propagating shapes for nGraph Function

`nGraph::Function` represents model with operations and their relations describing how data would flow through the function during the inference.
Data size estimations for output tensors of each operation is required for efficient model execution.
Such an estimation depends on the data type and shape of intermediate tensors of the model.
For OpenVINO supported operations you can find type and shape propagation rules in the documentation of the opset of your choice  

Shape propagation is a process of output shape calculation for each operation in the graph.
It starts from all the `Parameter` operations in the `ngraph::Function` and continues to calculate output shapes for all the operations of the `ngraph::Function` in the topological order.
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
`ngraph::PartialShape` stores shape values as a vector of `ngraph::Dimension` for the case if `ngraph::Rank` is static. 

`ngraph::Dimension` is represented by an interval which is a pair of values -- maximum and minimum value for the dimension, for both static and dynamic cases. 
They are equal for static dimensions and are set to different values for range and fully dynamic dimensions.

<pre>
1. dynamic ngraph::Dimension:
    ngraph::Dimension::dynamic();
    ngraph::Dimension(-1);  // special value for Dimension
    ngraph::Dimension{0, MAX_INT};
2. interval ngraph::Dimension:
    ngraph::Dimension{1, 10}; 
    ngraph::Dimension{0, MAX_INT};
3. static ngraph::Dimension
    ngraph::Dimension{3};
    ngraph::Dimension{3, 3};
</pre>

We allow creation of negative dimension on `ngraph::Interval`, `ngraph::Dimension`, `ngraph::PartialShape` level due to historical reasons, but there is no point in it in terms of shape propagation.
> NOTE: Dimension(-1) is equal to Dimension::dynamic(). There is no way to create static Dimension with value -1

## Shape propagation

The goal of shape propagation is to keep and pull all the known information about output shapes dimensions from the operation semantics.
During the shape propagation we may need input ranks/shapes for the operation or input values.
It is easy to lose detailed shape information. To provide practical guidance we have collected popular mistakes and the ways to avoid them:

Shape calculation DOs and DON`Ts:
1. Using `ngraph::Shape` class to work with shape instead of `ngraph::PartialShape` lead to losing detailed shape information
2. Calling `ngraph::PartialShape::to_shape()` method which converts `ngraph::PartialShape` object to `ngraph::Shape` object may result in exceptions thrown while running the code for dynamic `ngraph::Function`.
To avoid that introduce a check if the shape is dynamic before doing the conversion.
    ```
    ngraph::PartialShape shape = ...;
    if (shape.is_static())
    ```
    In order to make the code work for dynamic shapes too express all the needed shape calculations using capabilities of `ngraph::PartialShape`, `ngraph::Rank` and `ngraph::Dimension` classes.
3. To get the rank of the shape instead of `ngraph::Shape::size()` method do the following:
    ```
    ngraph::PartialShape shape = ...;
    ngraph::Rank = rank = shape.rank();
    if (rank.is_static())
        auto rank_value = rank.get_length();
        ...
    ```
4. Mathematical operators are overloaded for the `ngraph::Dimension` class and you can perform all kinds of manipulations on them.
It would result in correct calculations for both static and dynamic `ngraph::Dimension`. 
For example, multiplication of the first two dimensions of the shape is easy as:
```
ngraph::PartialShape shape = ...;
ngraph::Rank = rank = shape.rank();
if (rank.is_static() && rank.get_length() > 2)
    ngraph::Dimension product_of_first_and_second_dims = shape[0] * shape[1]; 
```
