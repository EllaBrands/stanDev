This page describes how users will use RStan 3.0 to compile and fit programs. This page is intended for RStan developers and developers of other user interfaces to Stan.

NB: These changes will take place in concert with the [[Stan C++ API Refactor|Stan Cpp API Refactor]].

## Overview / Example User Session
### Typical R  session
```R
# StanProgram() returns an instance of (reference) class StanProgram
> Program <- StanProgram({filename,program_as_string})  # if the .stan file does not parse this will fail quick. if not, this will be the "long" compile.
# Program$instantiate(data) returns an instance of StanProgramWithData (due to R reference classes lacking static/class methods)
> stan_program_with_data_instance <- Program$instantiate(data)  # this will be fast
> fit_object <- stan_program_with_data_instance$hmc(hmc_specific_parameters)
# alternatively: fit_object <- stan_program_with_data_instance$vb(rel_tol = 1e-4, abs_tol = 1e10)
# alternatively: optimize_result <- stan_program_with_data_instance$lbfgs(lbfgs_specific_parameters)
# alternatively: optimize_result <- stan_program_with_data_instance$optimize() # alias for $lbfgs() with only defaults
```
### Using the ProgramWithData instance
```R
> program_instance$log_prob(params)
```
### Using the Fit instance
```R
> fit_object$plot()
> fit_object$posterior_mean()
> fit_object$summary()
```
## Fit class specification
See the R documentation for [ReferenceClasses](http://stat.ethz.ch/R-manual/R-devel/library/methods/html/refClass.html)

**instance fields**
- param_draws [ params x chains x iterations ] : double
- param_names  params : string
- num_warmup 1 : long
- timestamps: timestamp (long) for each iteration (possibly roll into param_draws)
- mass matrix : NULL | params | params x params : double
- diagnostic draws: diagnostic params x chains x iterations
- diagnostic names: diagnostic params : string

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
