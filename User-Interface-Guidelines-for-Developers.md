This page describes how users will use RStan 3.0 to compile and fit models. This page is intended for RStan developers and developers of other user interfaces to Stan.

NB: These changes will take place in concert with the [Stan C++ API Refactor](/wiki/Stan-Cpp-API-Refactor).

## Overview / Example User Session
### Typical R  session
```R
> StanModelClass <- stan_compile({model_as_string, filename})  # if the .stan file does not parse this will fail quick. if not, this will be the "long" compile.
> model_instance <- StanModelClass$instantiate(data)  # this will be fast
> fit_object <- model_instance$hmc(hmc_specific_parameters)
# alternatively: fit_object <- model_instance$vb(rel_tol = 1e-4, abs_tol = 1e10)
# alternatively: optimize_result <- model_instance$lbfgs(lbfgs_specific_parameters)
# alternatively: optimize_result <- model_instance$optimize() # alias for $lbfgs() with only defaults
```
### Using the model instance
```R
> model_instance$log_prob(params)
```
### Using the fit instance
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

## Model class specification
**instance fields**
- stan_code
- cpp_code
- dso: <S4 cxxdso> (from inline)
- cxx_flags

**instance (reference class) methods**
- double log_prob(unconstrained_params)
- log_prob_grad(unconstrained_params)
- hessian(unconstrained_params)
- laplace_approx(unconstrained_params)
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
- cache models if possible

## Previous discussion
- [Stan-3-Unified-Interface](https://github.com/stan-dev/stan/wiki/Stan-3-Unified-Interface)
- [Interfaces-3.0-Spec](https://github.com/stan-dev/rstan/wiki/Interfaces-3.0-Spec)