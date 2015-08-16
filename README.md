# TensorContraction

[![Build Status](https://travis-ci.org/andyferris/TensorContraction.jl.svg?branch=master)](https://travis-ci.org/andyferris/TensorContraction.jl)

A package for implementing simple tensor operations in a user-friendly manner. A tensor is a decorated multi-dimensional array, with a name attached to each index. In a tensor contraction (generalized matrix multiplication), contracted indices are automatically mathced.

A simple example is:

```julia
v = [1 2 3 4]
M = [x*y for x=2:5,y=3:6]
@tensor M[[a,b]] * v[[b]]
```

This implements matrix-vector multiplication, resulting in a vector with a single index named `a`. 

Tensors can also be de-indexed in the desired order, on either side of the equality:

```julia
@tensor M = (rand(4,4))[[a,b]] # a rank-2 tensor
@tensor M[[b,a]] # the transpose of M (an Array)
@tensor M2[[b,a]] = M # M2 must already be of type Array{T,2}
```

### How it works

TensorContraction relies heavily on Julia's expressive type system. The index names are embedded as `Symbol` *values* like `Val{:a}` and are a part of the tensor's parametric type.

In this way, appropriate code is generated by `*`, `getindex` and `setindex!` at *compile time* and the operations run at full-speed at *run time*. 

The `@tensor` macro simply mangles the expression so that ``[[a,b]]`` is mapped into `[Val{:a},Val{:b}]` for readability purposes. 

### Limitations

1. Until Julia implements generated types (a.k.a. type-families), `TensorContraction` must implement each rank tensor as unique types, such as `Tensor1`, `Tensor2`, `Tensor3`, etc. These are defined until rank-32 (at which point memory requirements mean larger ranks might not have very much practical use). The resulting very large number of methods means the package is a little slow to load.

2. For full speed, ensure that the names of the indices are fully determined at compile-time so that the appropriate code is only generated once and so that function *specialization by value* does not lead to performance degradation.

3. Single-bracket indexing is still safe - use `hcat` and `vcat` when necessary to concatenate inside expresions like `@tensor v[vcat(indices1,indices2)][[a]]`.

 