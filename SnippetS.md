# SnippetS
This document describes the design and rationale for snippets code generator. Implementation of code functionality is located [here](https://github.com/openvinotoolkit/openvino/tree/master/inference-engine/src/snippets). Proposal for CPU backend integration is [here](https://github.com/openvinotoolkit/openvino/pull/2824).

# Rationale

We believe that core **CNN operators (convolution, gemm, fully connected) are limited by compute, the rest is memory bound**. Math approximations (like transcendental functions) are rare in emerging workloads and could be treated with the same machinery. **Snippets are designed to optimize topology for memory**, while leaving compute intensive kernels for backend developers.

We believe **potential speedup is proportional to shrink in memory-walked bytes**. So we can transform the problem to a task to optimize for memory walks, whatever pattern snippet has and operations it contains. Number of memory walks should be less or equal to handcrafted optimizations. This guarantees performance improvements over the previous approach (excluding corner cases caused by cache effects). *Shrinkage factor might be encoded to some cost function in future evolution of code generator*. Snippets generator provides diagnostics to estimate this shrinkage factor with `ngraph::snippets::op::Subgraph::print_statistics(bool verbose)` member.

We design SnippetS generator for back-end developers. The main purpose of inventing snippets code generator is an **operator fusion**, **register allocation** and **target kernel generation** decomposition. This allows modifications (like new fusion support) and feature extensions (like new operation support) to be done in a single point of modification and avoid combinatorial explosion for fusions/types/architectures etc.

We believe that creating a full-fledged compiler or usage of existing compiler infrastructure (like LLVM & MLIR) is superfluous at this point of evelition. We aim to provide a **flexible and performant framework for operation fusions**, leaving micro optimizations (e.g. instruction scheduling) to the backend H/W.

We do not aim to invent a DSL for SnippetS and would like to keep it this way. DSL gives users more flexibility to express uncommon operations. However, the shift towards an approach to encode topologies with elementary operations followed by smart enough fusions is already expressive and performant enough.

**Snippet** is a compiled compute **kernel** generated from a subgraph using SnippetS code generator for specific architecture with a **scheduling domain**. Using this scheduling domain and calling convention backend can execute generated compute kernels. For the first generation, snippets are **statically scheduled towards the output domain**. Multi-output snippets are supported if all outputs are broadcast-compatible in a sense that domains for all outputs can be broadcasted from one root domain which defines snippet schedule. It’s a subject of extension for future generations.

We use nGraph as the highest level IR for subgraph representation and lowering transformations. **Opset1** is a base operation set for code generation. We aim to **keep the minimal possible and sufficient operation set** (or ISA) and keep it **RISC-like** (memory and compute decomposed).

**One subgraph corresponds to one snippet**. Operations which cannot be scheduled by a single schedule should not be placed in the same subgraph. Snippet somewhat conceptually close to OpenCL kernel without a restriction to express only embarrassingly parallel tasks.
**Subgraph** once extracted from full topology IR is **treated as an operation and data flow descriptor in scalar notation** (similar to OpenCL/CUDA). Tensor sizes are used for defining scheduling domain and detecting broadcasts/reductions.

We split operations into 3 groups: **layout-oblivious (LOO), layout-aware(-tolerant) and layout-dependent**. **Layout-oblivious** operation semantics and implementation are completely agnostic to a specific layout in which tensors are placed in memory. For example, elements-wise math and ReLU does in this category. Implementation **layout-aware** operation depends on the layout of input/output tensors. For example, convolutions and other block-wise kernels or layout repaks. For **layout-specific** operation semantics and implementation depends on the layout. For example, the Yolo region. Patterns to fuse constructed in terms of taxonomy above.

# Design

Code generation is split into 2 phases, **tokenization** and **lowering**.

## Tokenization

Tokenization runs on full topology nGraph function inside a specific plugin in a stage of common transformations. Input of tokenization is a topology graph. Output is a modified topology graph with `ngraph::snippets::op::Subgraph` operations installed. Each subgraph contains nGraph function (called **body**) which holds a part of original topology legal for snippet generation (can be scheduled with a single schedule) 

Procedure of finding subgraphs suitable for code generation is called **tokenization**, meaning that we split the topology tree into subgraphs in the same greedy approach which is used for parsing input stream of characters into the tokens. It also could be seen as and modified into a basic block construction problem, since we also find a leader and potentially terminators. Implementation can be found [here](https://github.com/openvinotoolkit/openvino/blob/master/inference-engine/src/snippets/src/pass/collapse_subgraph.cpp).

Tokenization has an advantage over the pattern matching approach (used in traditional and MLIR-based compilers) since it can handle arbitrary patterns of operations. Pattern matching deduces specific configuration of operations to translate to another one, more suitable for target machine or further lowering. This means that relations between operations are fixed. Tokenization on the other hand has the only limitation on specific operation types which are **suitable and profitable** to fuse with respect to original topology correctness (keeping it as a direct acyclic graph).

The extracted body comes to a plug-in wrapped as a composite `Subgraph` operation which is seen as a block box from a plugin standpoint and can participate in any plugin specific subroutines (e.g. layout assignment, memory allocation, etc.).

## Supported subgraph patterns

Subgraph accepts arbitrary numbers of inputs and outputs. There is 1:1 mapping for external (subgraph node’s) and internal (body) parameters indexes. 

Pattern here is an exact subgraph configuration (nodes and edges between them). **The first generation of snippets supports only layout-oblivious operations which may have broadcast on inputs and broadcast-compatible outputs**. For example Shapes `<1, 42, 17, 31>`, `<1, 42, 17, 1>` and `<1, 42, 1, 31>` are considered as broadcast-compatible. Layout-oblivious operation with multiple outputs as a snippet leader and forms a new subgraph. The most beneficial patterns are subgraphs with complex control flow but minimal number of inputs/and outputs. For example, GeLU has a 5x shrinkage factor from original unfused subgraph in number of bytes walked. Subgraph below could be considered as an example of such a subgraph. Leader detection procedure aims to find such subgraphs.

```
            ...
             |
            Add
             |
     +-------+-------+
     |               |
     |              Add
     |               |
     |             Clamp
     |               |
     +---Multiply----+
             |
            ...
```

Operations are greedily added to the subgraph until
1. New operation doesn’t introduce a loop in a topology function.
1. Number of inputs and outputs satisfies target criteria.
1. Operation is not a predecessor of topology output.
1. Resulting subgraph can be scheduled (all outputs are broadcast-compatible). 

If a potential subgraph doesn’t meet any of criteria above, the procedure continues to find a new leader.

## Lowering

Lowering is a sequence of subgraph (snippet body) traversal passes to generate a compute kernel out of subgraphs of operations extracted by tokenization.

1. Common optimizations
1. Canonicalization for snippets dialect
1. Target-specific optimizations
1. Register allocation
1. Schedule generation
1. Target code emission



