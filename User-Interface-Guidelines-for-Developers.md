This page describes how users will use Stan 3.0 to compile and fit programs. This page is intended for developers of any user interfaces to Stan.

These changes will take place in concert with the [[Stan C++ API Refactor|Stan Cpp API Refactor]]. Also, the [R implementation](https://github.com/stan-dev/rstan/blob/develop/rstan3/R/AllClass.R) has tentatively started.

Previous discussion
- [Stan-3-Unified-Interface](https://github.com/stan-dev/stan/wiki/Stan-3-Unified-Interface)
- [Interfaces-3.0-Spec](https://github.com/stan-dev/rstan/wiki/Interfaces-3.0-Spec)


## Overview / Example User Session
In general, the basic steps are:

1. Compile the C++, which will become more demanding once we start using algorithms that rely on fvars
2. Instantiate it. Bob thinks that we need to have an abstract base class for programs to inherit from, rather than templating the program. It is not yet clear what all the public methods of the abstract base class will be.
3. Call one of the algorithms Stan supports to estimate the model. The output is to be handled by a VarWriter that can be implemented and passed in by the interface.
4. Diagnose

## Major Unresolved issues

* To what extent should the steps in the interfaces mirror those of the C++ / CmdStan?
* What should be stand-alone functions and what should be methods, as in `build(config)` vs. `config$build()`?
* Do we need a separate class that only exposes the algorithms in the C++ library or is it better to for the class to expose both algorithms and lower-level functions like `log_prob`?
* How granular should the estimation functions be, as in `$hmc_nuts_diag_e_adapt()` vs. `$hmc(adapt = TRUE, diag = TRUE, metric = "Euclidean")`

### Typical R session
We plan to use [ReferenceClasses](http://stat.ethz.ch/R-manual/R-devel/library/methods/html/refClass.html) throughout. See the examples section of [this](https://github.com/stan-dev/rstan/blob/develop/rstan3/R/rstan.R) for a canonical R example.

### Typical PyStan session
WIP
```python
# Step 1 --- Create a StanProgram *class*
import pystan
MyStanProgramClass = pystan.compile(program_code=code, program_name=name)

# Step 2 --- Create an instance of StanProgram (with data, equivalent to StanProgramWithData-class object)
program = MyStanProgramClass(data)

# Step 3 --- Estimate the parameters
estimates = dprogram.optimize()       # maximum a posteriori estimator
estimates = dprogram.ehmc(delta=.9) # MCMC from the posterior distribution
 
# Diagnose any problems
# TODO

# sampling (default) then extract
fit = dprogram.sampling()
alpha = fit['alpha']
```

### Typical CmdStan session

TBD but Julia and MATLAB just call CmdStan and thus would be similar.

### Typical StataStan session

TBD but StataStan also calls CmdStan but would probably have some quirks

## StanConfig

### RStan

* program.name  (like "8schools")
* shared.object (path to the shared object on disk)
* a field for each thing declared in the `data` block of a Stan program

### PyStan

TBD, basically the same

## StanModule

### Methods provided by the Stan library to all interfaces

The class would expose the following methods from the abstract base class (that needs to be implemented):

- scalar log_prob(unconstrained_params)
- vector grad(unconstrained_params)
- tuple  log_prob_grad(unconstrained_params)
- matrix hessian(unconstrained_params)
- tuple  log_prob_grad_hessian(unconstrained_params)
- scalar laplace_approx(unconstrained_params)
- vector constrain_params(unconstrained_params = <vector>)
- vector unconstrain_params(constrained_params = <vector>)
- tuple  params_info() would return
    - parameter names
    - lower and upper bounds, if any
    - parameter dimensions
    - declared type (cov_matrix, etc.)
    - which were declared in the parameters, transformed parameters, and generated quantities blocks
 
It is not clear if the following algorithms would be methods, methods of a different class, or stand-alone functions that input a program and configuration options:

- tuple sample()
- tuple advi()
- tuple maximize()

## MCMC output containers

### RStan

The VarWriter would fill an Rcpp::NumericVector with appropriate dimensions. Then there would be a new (S4) class that is basically an array to hold MCMC output for a particular parameter, which hinges on the params_info() method. The (array slot of a) StanType has dimensions equal to the original dimensions plus two additional trailing dimensions, namely chains and iterations. Thus,
- if the parameter is originally a scalar, on the interface side it acts like a 3D array that is 1 x chains x iterations
- if the parameter is originally a (row) K-vector, on the interface side it acts like a 3D array that is K  x chains x iterations
- if the parameter is originally a matrix, on the interface side it acts like like a 4D array that is rows x cols x chains x iterations
- if the parameter is originally a ``std::vector`` of type ``foo``, on the interface side it acts like a multidimensional array with dimensions equal to the union of the ``std::vector`` dimensions, the ``foo`` dimensions, chains, and iterations

The advantages of having such a class hierarchy are
- can do customized summaries; i.e. a ``cov_matrix`` would not have its upper triangle summarized because that is redundant with the lower triangle and we can separate the variances from the covariances and a ``cholesky_factor_cov`` would not have its upper triangle summarized because its elements are fixed zeros
- can easily call unconstrain methods (in C++) for one unknown rather than having to do it for all unknowns
- can implement probabalistic programming; i.e. if ``beta`` is a estimated vector in Stan, then in R ``mu <- X %*% beta`` is N x chains x iterations and if ``variance`` is a scalar then ``sigma <- sqrt(variance)`` is a can be summarized appropriately (including n_eff, etc.)

### PyStan

TBD. Allen is ambivalent about doing something like a StanType class hierarchy

### CmdStan

Everything gets written to a flat CSV file

## MCMC output bundle

### StanFitMCMC class in R

TBD

### StanFitMCMC class in PyStan

**instance fields**
- param_draws : This needs to be [ params x chains x iterations ] : double in contiguous memory or similar if the C++ API for split_rhat is going to be called directly and fast.
- param_names  params : string
- num_warmup 1 : long
- timestamps: timestamp (long) for each iteration (possibly roll into param_draws)
- mass matrix : NULL | params | params x params : double
- diagnostic draws: diagnostic params x chains x iterations 
- diagnostic names: diagnostic params : string


## Optimize output bundle

### StanFitOptimize
note: computation of the hessian is optional at optimization time
**instance fields**
- param values
- param names
- value (value of function)
- (optinal) hessian
- (optional) diagnostic param values: diagnostic params x iterations
- diagnostic param names: diagnostic params : string

## Recommendations and style guidelines
- consistent variable names (either ``num_warmup`` and ``num_iter`` or use ``warmup`` and ``iter``, not a mix)
- at the start of sampling, the (default) config for the chosen sampling algorithm should be displayed with helpful tips for adjustments (perhaps using color? follow clang's color strategy)
- cache programs if possible

