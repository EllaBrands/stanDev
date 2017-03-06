# Introduction

Currently Stan does not allow for within-chain parallelism due to

1. design of the AD stack
2. requirement to run on all major platforms (including Windows)
3. inability of most of our users (laptop) hardware to run more than 4 processes at once.

This can become a major bottleneck whenever many expensive operations are required to defined the log-density. Of course, counter-argument 3 is always a given, but as Stan is used more and more on large problems which are usually solved on computer clusters, we will dismiss it in the discussion here.

A very common situation for Stan models are hierarchical models which require the evaluation of the per-group mean for group-specific parameter sets. So consider the situation when we have g=1...G groups and each group has a parameter vector theta_g of length P. A good example are longitudinal hierarchical models where the group mean is defined by an ODE which must be solved using our stiff or non-stiff ODE solver for each group at the time-points ts at which we have data observed.

Right now, a Stan program will have to call the `integrate_ode` function for each group separately in serial order. The ODE integration for all groups will dominate the overall cost to calculate the gradient of the log-density simply by the enormous numerical cost implied by the ODE integration process.

Since hierarchical models are so common in Stan and many models evolve around a repeated evaluation of some expensive function this page aims to discuss and collect ideas to solve this. The key ideas are

- Augment the Stan language with specialized `_parallel` functions in addition to the base function
- The `_parallel` function must not use AD to calculate the gradients, but instead use analytic only expression. The analytic expressions are calculated in parallel and then merged back into the AD graph.
- Use OpenMP which is a cross-platform compiler solution allowing simple parallelizations of for loops. Compilers not supporting OpenMP will still compile the program, but no parallelization will occur.

We will use the ODE case as an example below, but I would assume (hope) that other subsystems in Stan can facilitate this approach (GPs)?

# Notation

- `g=1...G` groups
- theta_g: P parameters per group g
- Theta: GxP 2D array of parameters
- t0_g, y0_g: initial time-point and initial state of the ODE
- T0, Y0: array of initials of all G groups
- ts_g: time-series array per group g
- Ts: concatenated time-series vector of all groups [ts_1, ..., ts_G]
- S_Ts: Slicing index vector which contains [size(ts_1), ..., size(ts_G)]
- x_r_g: auxiliary real data for group g
- X_r: concatenated auxiliary real data [x_r_1, ..., x_r_G]
- S_X_r: slicing vector for X_R [size(x_r_1), ..., size(x_r_G)]
- x_i_g, X_i, S_X_i: same for int auxiliary data

# Stan Language

The basic idea is to add to the Stan language parallel versions of particularly expensive functions. This implies that functions which were previously called for each group with data specific to the given group now need to be called with all data specified at once. The `integrate_ode` function currently has the signature

```C++
real [,] integrate_ode(ode_functor,
                       real[] y0_g, real t0_g,
                       real[] ts_g,
                       real[] x_r_g,
                       int[] x_i_g)
```

The `integrate_ode_parallel` function now has to take all data at once for all groups and as we do not have ragged arrays in the Stan language (yet), we need to add respective slicing index vectors:

```C++
real [,] integrate_ode_parallel(ode_functor,
                                real[,] Y0, real[] T0,
                                real[] Ts, int[] S_Ts,
                                real[] X_r, int[] S_X_r,
                                int[] X_i, int[] S_X_i)
```

So the suggestion here is to increase the dimensionality of the input data structures accordingly and add a slicing variable where needed.

# Parallel ODE Prototype

A very simple toy example of the above ideas are coded on a branch on stan-math:

https://github.com/stan-dev/math/blob/feature/proto-parallel-ode/test/unit/math/rev/mat/functor/integrate_ode_parallel_proto_test.cpp

This version is not as fancy in terms of slicing, but it has all the elements in place which are needed. In first tests of the parallel ODE code on a real project lead to substantial speedups from 3:30h on 1 core to 20min on 16cores. This prototype required some (minor) changes to the ODE subsystem for which I will start a refactoring soon. The specific points to be addressed in the ODE refactor are

1. Harmonize stiff and non-stiff subsystem
2. Allow for a user-specified Jacobian of the ODE RHS


