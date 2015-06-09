This page describes how users will use RStan 3.0 to compile and fit programs. This page is intended for RStan developers and developers of other user interfaces to Stan.

NB: These changes will take place in concert with the [[Stan C++ API Refactor|Stan Cpp API Refactor]]. Also, the [R implementation](https://github.com/stan-dev/rstan/blob/develop/rstan3/R/AllClass.R) has tentatively started.

## Overview / Example User Session
### Typical R  session
```R
# StanProgram() returns an instance of (reference) class StanProgram
> program <- StanProgram({filename,program_as_string})  # compilation step, time-consuming
> program_with_data <- program$instantiate(data)  # fast, returns instance of StanProgramWithData
> fit_object <- program_with_data_instance$hmc(hmc_specific_parameters)
# alternatively: fit_object <- program_with_data_instance$vb(rel_tol = 1e-4, abs_tol = 1e10)
# alternatively: optimize_result <- program_with_data_instance$lbfgs(lbfgs_specific_parameters)
# alternatively: optimize_result <- program_with_data_instance$optimize() # alias for $lbfgs() with only defaults
```
### Using the ProgramWithData instance
```R
> program_with_data$log_prob(params)
```
### Using the Fit instance
```R
> fit_object$plot()
> fit_object$posterior_mean()
> fit_object$summary()
```
## StanParameter class specification
This would be a new (S4) class that is basically an array to hold MCMC output for a particular parameter. Thus, the C++ library needs to be able to indicate
- whether an unknown is a parameter, transformed parameter, generated quantity, or diagnostic (e.g. ``accept_stat__``)
- what the dimensions of each unknown are (or if it is a scalar)
- what output corresponds to what unknown

On the interface side, the (array slot of a) StanParameter has dimensions equal to the original dimensions plus two additional trailing dimensions, namely chains and iterations. Thus,
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

In R, there is not much distinction between a vector and a one-column matrix or between a vector and a row vector, so the inheritance is R-based rather than Stan-based. PyStan might implement it a bit differently.


## Fit class specification
See the R documentation for [ReferenceClasses](http://stat.ethz.ch/R-manual/R-devel/library/methods/html/refClass.html)

**instance fields**
- param_draws : named list (R) or dict (Python) where each element is a ``StanParameter`` (AR: I think this needs to be [ params x chains x iterations ] : double in contiguous memory or similar if the C++ API for split_rhat is going to be called directly and fast. Could users access the draws indirectly -- e.g., via a layer of indirection a la the extract method?)
- param_names  params : string (Ben thinks this is unnecessary given the names of param_draws)
- num_warmup 1 : long
- timestamps: timestamp (long) for each iteration (possibly roll into param_draws)
- mass matrix : NULL | params | params x params : double
- diagnostic draws: diagnostic params x chains x iterations (Ben thinks this is unnnecessary given the type of each ``StanParameter`` in param_draws)
- diagnostic names: diagnostic params : string (Ben thinks this is unnecessary given the above)

**instance (reference class) methods**
- show (minimal)
- summary (exhaustive)
- posterior_mean (was: get_posterior_mean)

## StanProgram class specification
**instance fields**
- file: character
- stan_code: character
- cpp_code: character
- dso: S4 cxxdso (from inline) which includes cxx_flags as a slot

The ``file`` slot is somewhat redundant because ``stan_code`` and ``cpp_code`` could be inferred from it. However, ``file`` is necessary for the ``save`` method because we want to save the serialized version in the same directory by default. But if the user initializes with a string defining the Stan program rather than a string defining the path to a Stan program, then the string is first written to a file in the temporary directory. In that case, if the serialized object is loaded in a new session, then the temporary file will not exist, in which case we need to have serialized the object with the ``stan_code`` and ``cpp_code``.

## StanProgramWithData class specification
**instance (reference class) methods**
- double log_prob(unconstrained_params)
- vector grad(unconstrained_params)
- list log_prob_grad(unconstrained_params)
- matrix hessian(unconstrained_params)
- list log_prob_grad_hessian(unconstrained_params)
- double laplace_approx(unconstrained_params)
- vector constrain_params(unconstrained_params = <vector>)
- vector unconstrain_params(constrained_params = <vector>)
- stan_fit hmc(hmc_params)
- stan_fit vb(vb_params)
- stan_fit sample # alias for default sampler
- optimize_result lbfgs(lbfgs_params)
- optimize_result newton(newton_params)
- stan_fit optimize # alias for default optimizer

## Optimize class specification
note: computation of the hessian is optional
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

## Previous discussion
- [Stan-3-Unified-Interface](https://github.com/stan-dev/stan/wiki/Stan-3-Unified-Interface)
- [Interfaces-3.0-Spec](https://github.com/stan-dev/rstan/wiki/Interfaces-3.0-Spec)
