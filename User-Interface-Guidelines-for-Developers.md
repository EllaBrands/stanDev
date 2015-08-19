This page describes how users will use Stan 3.0 to compile and fit programs. This page is intended for developers of any user interfaces to Stan.

These changes will take place in concert with the [[Stan C++ API Refactor|Stan Cpp API Refactor]]. Also, the [R implementation](https://github.com/stan-dev/rstan/blob/develop/rstan3/R/AllClass.R) has tentatively started.

Previous discussion
- [Stan-3-Unified-Interface](https://github.com/stan-dev/stan/wiki/Stan-3-Unified-Interface)
- [Interfaces-3.0-Spec](https://github.com/stan-dev/rstan/wiki/Interfaces-3.0-Spec)


## Overview / Example User Session
In general, the basic steps are:

1. Instantiate the templates, which will become more demanding once we start using algorithms that rely on fvars
2. Construct an instance of the model by passing the data to it. Bob thinks that we need to have an abstract base class for models to inherit from, rather than templating the model. It is not yet clear what all the public methods of the abstract base class will be.
3. Call one of the algorithms Stan supports to estimate the model. The output is to be handled by a VarWriter that can be implemented and passed in by the interface.

### Typical R session
We plan to use [ReferenceClasses](http://stat.ethz.ch/R-manual/R-devel/library/methods/html/refClass.html) throughout. See the bottom of [this](https://github.com/stan-dev/rstan/blob/develop/rstan3/R/rstan.R#L255) for the canonical example
```R
# Step 1 --- Create an object of StanProgram-class
program <- StanProgram(code = mc)     # preferable to specify a file
                                           
# Step 2 --- Create a StanProgramWithData-class object
dprogram <- program$instantiate()
 
# Step 3 --- Estimate the parameters
estimates <- dprogram$optimize()       # maximum a posteriori estimator
estimates <- dprogram$ehmc(delta = .9) # MCMC from the posterior distribution
 
# Step 4 --- Diagnose any problems
```

### Typical PyStan session
WIP
```
# Step 1 --- Create a StanProgram *class*
MyStanProgram = pystan.compile(code)

# Step 2 --- Create an instance of StanProgram (with data, equivalent to StanProgramWithData-class object)
dprogram = MyStanProgram(data)

# Step 3 --- Estimate the parameters
estimates = dprogram.optimize()       # maximum a posteriori estimator
estimates = dprogram.ehmc(delta=.9) # MCMC from the posterior distribution
 
# Step 4 --- Diagnose any problems
```

### Typical CmdStan session

TBD but Julia and MATLAB just call CmdStan and thus would be similar.

### Typical StataStan session

TBD but StataStan also calls CmdStan but would probably have some quirks

## StanProgram

### RStan

**instance fields**
- stan_code: character
- cpp_code: character
- dso: S4 cxxdso (from inline) which includes cxx_flags as a slot

### PyStan

TBD

## StanModelWithData

### Methods provided by the Stan library to all interfaces

The model class would expose the following methods from the abstract base class:

- scalar log_prob(unconstrained_params)
- vector grad(unconstrained_params)
- tuple  log_prob_grad(unconstrained_params)
- matrix hessian(unconstrained_params)
- tuple  log_prob_grad_hessian(unconstrained_params)
- scalar laplace_approx(unconstrained_params)
- vector constrain_params(unconstrained_params = <vector>)
- vector unconstrain_params(constrained_params = <vector>)

The model class would expose the following additional methods
- tuple  params_info() would return
    - parameter names
    - lower and upper bounds, if any
    - parameter dimensions
    - declared type (cov_matrix, etc.)
    - which were declared in the parameters, transformed parameters, and generated quantities blocks
 
It is not clear if the following algorithms would be methods or stand-alone functions that input a model and configuration options:

- tuple [a-z]+hmc()
- tuple lbfgs()
- tuple bfgs()
- tuple newton()
- tuple vb()

### R methods for the StanProgramWithData instance
Hopefully, we can use [exposeClass()](http://www.inside-r.org/packages/cran/rcpp/docs/exposeClass) with [setRcppClass()](http://www.inside-r.org/packages/cran/rcpp/docs/setRcppClass) to expose Stan's abstract base class for models as an (internal) ReferenceClass in RStan at build time and then inherit a (public) ReferenceClass from that when the data are passed in at run time. For example,
```R
> dprogram$log_prob(params)
> dprogram$ehmc(chains = 8)
> dprogram$optimize() # alias for default, no arguments allowed
> dprogram$sample()   # alias for default, no arguments allowed
```

### PyStan methods for the StanProgramWithData instance

TBD

## MCMC output containers

### RStan

The VarWriter would fill an Rcpp::NumericVector with appropriate dimensions. Then there would be a new (S4) class that is basically an array to hold MCMC output for a particular parameter, which hinges on the params_info() method. The (array slot of a) StanParameter has dimensions equal to the original dimensions plus two additional trailing dimensions, namely chains and iterations. Thus,
- if the parameter is originally a scalar, on the interface side it acts like a 3D array that is 1 x chains x iterations
- if the parameter is originally a (row) K-vector, on the interface side it acts like a 3D array that is K  x chains x iterations
- if the parameter is originally a matrix, on the interface side it acts like like a 4D array that is rows x cols x chains x iterations
- if the parameter is originally a ``std::vector`` of type ``foo``, on the interface side it acts like a multidimensional array with dimensions equal to the union of the ``std::vector`` dimensions, the ``foo`` dimensions, chains, and iterations

The advantages of having such a class hierarchy are
- can do customized summaries; i.e. a ``StanCovMatrix`` would not have its upper triangle summarized because that is redundant with the lower triangle and we can separate the variances from the covariances and a ``StanCholeskyFactorCov`` would not have its upper triangle summarized because its elements are fixed zeros
- can easily call unconstrain methods (in C++) for one unknown rather than having to do it for all unknowns
- can implement probabalistic programming; i.e. if ``beta`` is a ``StanVector`` then ``mu <- X %*% beta`` is a ``StanVector`` but whose dimensions are N x chains x iterations and if ``variance`` is a ``StanReal`` then ``sigma <- sqrt(variance)`` is a ``StanReal``, which can be summarized (including n_eff, etc.)

The class tree looks like
- StanParameter: a virtual class with slots for name (character), theta (array), and type (character among {unconstrained parameter, constrained parameter, transformed parameter, generated quantity, diagnostic})
    - StanReal
        - StanInteger which can only be a generated quantity or diagnostic
            - StanFactor which has a levels slot to indicate the categories corresponding to the integer draws
    - StanMatrix
        - StanCovMatrix
            - StanCorrMatrix
        - StanCholeskyFactorCov
            - StanCholeskyFactorCorr
        - StanVector
            - StanRowVector
            - StanSimplex
            - StanUnit

In R, there is not much distinction between a vector and a one-column matrix or between a vector and a row vector, so the inheritance is R-based rather than Stan-based.

### PyStan

TBD. Allen is ambivalent about doing something like a StanParameter class hierarchy

### CmdStan

Everything gets written to a flat CSV file

## MCMC output bundle

### StanFitMCMC class in R

**instance fields**
- warmup_draws:  named list where each element is a ``StanParameter``
- sample_draws : named list where each element is a ``StanParameter``
- timestamps: timestamp (long) for each iteration (possibly roll into param_draws)
- mass matrix : NULL | params | params x params : double

```R
> estimates$plot()     # or $pairs(), $traceplot(), etc.
> estimates$posterior_means() # possibly accumulate posterior_variances too
> estimates$show()     # minimal
> estimates$summary()  # exhaustive 
```

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

