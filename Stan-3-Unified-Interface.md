I'd very much like to unify the look and feel of PyStan, CmdStan, and RStan to the extent possible.  Some related Wiki pages:

* [Services refactor in Stan C++](https://github.com/stan-dev/stan/wiki/Stan-API-Refactor)  [stan wiki]
* [RStan 3 refactor](https://github.com/stan-dev/rstan/wiki/Interfaces-3.0-Spec)  [rstan wiki]

I'm going to write this primarily from an RStan perspective, but do not want to lose sight of the fact that the goal is consistency of functionality and naming across the interfaces.

## Compilation

The compilation function compiles a Stan program to a dynamic shared object (DSO) and optionally saves it.  

```
compiled_program
rtan::stan_compile(                # DEFAULTS
  source = (<string> || <path>)    # no default
  name = <string>,                 # empty
  boost = <path>,                  # use BH
  eigen = <path>,                  # use RcppEigen
  opt = ("fast" || "debug"),       # "fast"
  save_dso = <bool>,               # FALSE
  debug_code = <bool>              # TRUE
);
```

* Is there any reason to keep the name option?
* How long-lived is the `save_dso` option?
    - Where is it saved?
* If we allow optimize settings, is there a way to have it scope over just the call and not affect the R global build environment?
* Is there a way we can have make-like behavior here where if there's a saved version and the model file or code have not changed, don't do anything?
* Do we need the other existing compiler options (`"presentation1"`, `"presentation2"`, and `"small"`)?
* Should the save_dso provide a path rather than a boolean?
* Is it possible to save the DSO across sessions?
* Should the dave_dso defalt be true and if so, use temp?
* Do we ever need `NDEBUG=false`?  That is, shouldn't we just always have `-DNDEBUG` set to turn off debugging?

## Compiled Program Object

This should only include the compiled Stan program and methods to do variable conversion, extract names, compute log densities, etc.

```
Slots:
  name : <string>
  stan_code : <string>
  cpp_code  : <string>
  dso : <S4 cxxdso>  (from inline)
  cxx_flags: <string>
Methods:
  double log_prob(
    unconstrained_params = <vector>,
    jacobian = <bool>,
    list(value=<double>, 
         grad=<vector>);

  log_prob_grad(
   unconstrained_params = <vector>,   
   jacobian = <bool>);

  return$value : <double>
  return$grad : <vector>

 list(
 constrain_params(
   unconstrained_params = <vector>);

 vector
 constrain_params_vec(
   unconstrained_params = <vector>);

 vector
 unconstrain_params(
   params = (<list> || vector));
```

* Seems there should be a better way to organize this, but I can't see it.  And maybe some shorter names


## NUTS

```
mcmc_fit
rstan::stan_nuts(
  program = <compiled_program>,       # no default
  data = (<list> || <environment>),   # top-level environment by default
  max_treedepth = <int>,              # 10
  chain_ids = <ints>,                 # 1:4
  num_warmup = <int>,                 # 10
  num_samples = <int>,                # 10
  init=(<double>                      # 2.0
        || <list>
        || <list of lists>
        || <function>)
  stepsize=<double>,                  # 1.0
  metric = ("unit_e",                 # "diag_e"
            || "diag_e"
            || "dense_e"
            || <vector>
            || <matrix>),                  
  adapt_stepsize = <bool>,            # TRUE
  adapt_mass_matrix = <bool>,         # TRUE
  adapt = list(gamma=<double>,
               kappa=<double>,
               t0=<double>,
               init_buffer=<int>,
               term_buffer=<int>,
               adapt_window=<int>)    # ...
  params_to_save = (NA 
                    || <vector(char)>) # NA
  save_warmup = <bool>,               # TRUE
  thin = <int>,                       # 1
  sample_file = <path>,               # NA
  diagnostic_file = <path>,           # NA
  refresh = <int>,                    # 1
  verbose = ("quiet" 
             || "standard"
             || "verbose"),           # standard
  seed = <int>                        # random from R
)
```

* Return value just includes the result in such a way that it can be extracted as before.
* Return value won't include the compiled program (or source).
* Should return value include timing information?  
* There's no `append_samples = <bool>`
* There's no option to compile the model from within the `stan_nuts` call.
* data defaults to top-level environment with lazy read (not how it's done now);  list is list of values as before
* Instead of `chains` and a start ID, there's just a `chain_ids` argument supplying both the chains and the ID, defaulting to the usual
* Default `num_warmup` and `num_samples` very small by default (but shouldn't be so small as to trigger scary messages about adaptation)
* Metric is specified by name of type or by value;  the names default to unit values for init
* Those adapt parameters are all for adapt_stepsize, right?
* Are there adaptation parameters for mass_matrix, like regularization?
* `params_to_save` defaults to `NA` (or `"all"`), or a positive list of whole parameter names.
* `sample_file` gets a copy of the output written to it, but return object still created --- should this just move so that the return object has a save on it?  Or should setting this cause the return of 
 this function to be `NA`?

#### MCMC Fit Object

## Euclidean HMC

Plain old HMC looks just like NUTS, but with max_treedepth replaced by integration time:

```
mcmc_fit
rstan::stan_hmc(
  ...
  int_time = <double>
  ...
)
```

This should share most of its implementation with `stan_nuts`, but be a separate command for simplicity.

## L-BFGS


```
optimization_fit
stan_lbfgs(
  program = <compiled_program>,
  data = (<list> || <environment>),
  init = (<double>
          || <list>
          || <list of lists>
          || <function>),
  optimization_file = <file>,
  verbose = ("quiet" || "verbose"),
  hessian = <bool>,                       # FALSE
  max_iterations = <int>,                 # NA
  save_iterations = <bool>,               # FALSE
  refresh = <int>,                        # 5
  init_alpha = <double>,                  # ???
  tol_obj = <double>,
  tol_grad = <double>,
  tol_param = <double>,
  tol_obj_rel = <double>,
  tol_grad_rel = <double>,
  tol_param_rel = <double>,
  seed = <int>
)
```

* Should `verbose` be a boolean?
* What does current `as_vector` do? Current doc says "flag indicating whether a vector is used to for the point estimate found. A list can be used instead by specifying it to be `FALSE`"
* What's the behavior of tolerances if more than one's specified?  I'd think it should be read as AND or as OR.
* No option to specify parameter return as vector --- use a compiled  model's conversion.

#### Optimization Fit Object

```
$par : parameters (as list)
$value : lp__
```

## Diagnose

```
diagnostics
rstan::stan_diagnose( 
 program = <compiled_program>,
 data = (<list> || <environment>),
 init = (<double>
         || <list>
         || <list of lists>
         || <function>),
 epsilon = <double>,
 error_threshold = <double>,
 seed = <int>,
 chain_ids = <int>)
```

* arguments just a subset of those for NUTS / HMC
* returns diagnostics for each of the chain_ids
* not sure what goes in diagnostics as long as it's easy to print

#### Diagnostics Object

Not sure what should go in here --- it needs to be printable.