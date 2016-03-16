# Design Document for the Services

## Overview

The current code wasn't designed. It just grew and grew. There have been heroic efforts to keep the code that controls Stan synchronized, but it's hard. The basic form of what's in CmdStan is duplicated to RStan and PyStan. Here's where our current development branches stand:

- [CmdStan](https://github.com/stan-dev/cmdstan/blob/develop/src/cmdstan/command.hpp): 894 lines
- RStan
  - [stan_fit.hpp](https://github.com/stan-dev/rstan/blob/develop/rstan/rstan/inst/include/rstan/stan_fit.hpp): 1598 lines 
  - [stan_args.hpp](https://github.com/stan-dev/rstan/blob/develop/rstan/rstan/inst/include/rstan/stan_args.hpp): 834 lines
- [PyStan](https://github.com/stan-dev/pystan/blob/develop/pystan/stan_fit.hpp): 1760 lines (stan_fit.hpp and stan_args.hpp in one file)

This isn't good.

### Why? What do we want to fix?

1. Maintenance.
2. Maintenance. 
3. Maintenance.

Software is hard. We've made it harder by duplciating code 3 times. Additionally, a lot of logic for running the different algorithms are inside those files and that makes fixing things difficult. It needs to be fixed in multiple places in order 

Not only is the code duplicated, there is logic for running the different algorithms inside the files and implementing it 3 times slightly differently is error prone. It's costing us too much time and effort to deal with the duplicated code.

## Scenarios

Here are three different scenarios that we will have to deal with.


### Scenario 1: Fixing a bug in an existing algorithm

We found a bug in an algorithm. We will need to update CmdStan, PyStan, and RStan with the fix. This fix should be made to the Stan Library. At the next release of Stan, the fix will be available to each of the repositories.

There should be minimal effort in CmdStan, RStan, and PyStan to change. Ideally, it would be none. In the current setup, we have to update CmdStan, RStan, and PyStan.

This will happen most frequently.

Current algorithms:

- sampling (13)
  - fixed parameter
  - static HMC
    - unit Euclidean metric
    - unit Euclidean metric with windowed adaptation
    - diagonal Euclidean metric
    - diagonal Euclidean metric with windowed adaptation
    - dense Euclidean metric
    - dense Euclidean metric with windowed adaptation
  - NUTS
    - unit Euclidean metric
    - unit Euclidean metric with windowed adaptation
    - diagonal Euclidean metric
    - diagonal Euclidean metric with windowed adaptation
    - dense Euclidean metric
    - dense Euclidean metric with windowed adaptation
- optimization (3)
  - LBFGS
  - BFGS
  - Newton
- ADVI (2)
  - mean field
  - full rank
- gradient test (1)

Total: 19


### Scenario 2: Adding a new algorithm

We create a new algorithm and want to expose it in CmdStan, RStan, and PyStan. The algorithm should be written in the Stan Library, fully tested, and stand-alone code. We should make it clear how to call the algorithm.

The effort to get a new algorithm to CmdStan, RStan, and PyStan should be low, but it will need to happen in all three places. We want the interface to the computing platform to be idiomatic to the platform. This requires work in creating calling conventions and arguments and shuttling them to the Stan Library's algorithm.

This will happen less frequently.


In addition to the list above, Michael believes we'll be adding:
- sampling (24)
  - static HMC
    - unit Riemannian metric
    - unit Riemannian metric with windowed adaptation
    - diagonal Riemannian metric
    - diagonal Riemannian metric with windowed adaptation
    - dense Riemannian metric
    - dense Riemannian metric with windowed adaptation
  - NUTS
    - unit Riemannian metric
    - unit Riemannian metric with windowed adaptation
    - diagonal Riemannian metric
    - diagonal Riemannian metric with windowed adaptation
    - dense Riemannian metric
    - dense Riemannian metric with windowed adaptation
  - Exhaustive HMC
    - unit Euclidean metric
    - unit Euclidean metric with windowed adaptation
    - diagonal Euclidean metric
    - diagonal Euclidean metric with windowed adaptation
    - dense Euclidean metric
    - dense Euclidean metric with windowed adaptation
    - unit Riemannian metric
    - unit Riemannian metric with windowed adaptation
    - diagonal Riemannian metric
    - diagonal Riemannian metric with windowed adaptation
    - dense Riemannian metric
    - dense Riemannian metric with windowed adaptation

Total (including other algorithms): 43

But perhaps we won't be adding and exposing all of these to users.

### Scenario 3: Updating defaults

We want to update an existing algorithm's default settings. We want this to be updated in CmdStan, RStan, and PyStan.

We want the effort to be minimal. Ideally, we could just change the Stan Library and have all the logic push through the interfaces.

This will happen less frequently. Probably less often than adding a new algorithm.


## Goals

The overall goal is maintenance. Here are some more concrete goals, some of which are competing:

- Make the Stan Library look more like idomatic C++. This will allow other developers to come into the project and be able to contribute.
- Lower the effort level to maintain CmdStan, RStan, and PyStan. Moving away from infrastructure discussions and working on real stuff would be better.
- Minimize the amount of code. This is just good practice. Being able to do more in less code is better.
- Keep the code simple. This is also good practice. A simpler, easier to understand solution is better than a more complex solution. This helps other developers with their ability to break down the code, fix it, and contribute.


## Proposed Solution #1: Daniel's solution

For each algorithm, the Stan Library has a function that can be called. For example, the function for NUTS, diagonal Euclidean metric with windowed adaptation will look something like this:

```C++
template <class Model, class rng_t>
int hmc_nuts_diag_e_adapt(Model& model,
                          rng_t& base_rng,
                          Eigen::VectorXd& cont_params,
                          int num_warmup,
                          int num_samples,
                          int num_thin,
                          bool save_warmup,
                          int refresh,
                          double stepsize,
                          double stepsize_jitter,
                          int max_depth,
                          double delta,
                          double gamma,
                          double kappa,
                          double t0,
                          unsigned int init_buffer,
                          unsigned int term_buffer,
                          unsigned int window,
                          interface_callbacks::interrupt::base_interrupt& interrupt,
                          interface_callbacks::writer::base_writer& sample_writer,
                          interface_callbacks::writer::base_writer& diagnostic_writer,
                          interface_callbacks::writer::base_writer& message_writer);
```

With the current set of algorithms, we would have 23 entry points that would have to be maintained and plumbed through to CmdStan, RStan, and PyStan. These entry points are simple to call from CmdStan, RStan, and PyStan and would look something like this:

- CmdStan:

```C++
...
if (engine->value() == "nuts" && metric->value() == "diag_e" && adapt_engaged == false) {
  stan::services::categorical_argument* base
    = dynamic_cast<stan::services::categorical_argument*>(algo->arg("hmc")->arg("engine")->arg("nuts"));

  int max_depth
    = dynamic_cast<stan::services::int_argument*>(base->arg("max_depth"))->value();
            
  return stan::services::sample::hmc_nuts_diag_e(model, base_rng, cont_params,
                                                 num_warmup, num_samples, num_thin,
                                                 save_warmup, refresh, stepsize,
                                                 stepsize_jitter, max_depth,
                                                 interrupt, sample_writer,
                                                 diagnostic_writer, info);
}
```

- RStan and PyStan look very similiar:

```C++
...
if (args.get_ctrl_sampling_algorithm() == "nuts" && metric == DIAG_E && args.get_ctrl_sampling_adapt_engaged() == false) {            
  return stan::services::sample::hmc_nuts_diag_e(model, base_rng, cont_params,
                                                 num_warmup, num_samples, num_thin,
                                                 save_warmup, refresh, stepsize,
                                                 stepsize_jitter, max_depth,
                                                 interrupt, sample_writer,
                                                 diagnostic_writer, info);
}
```

A lot of the boilerplate code within the interfaces could be greatly simplified if we were able to go with flat entry points. 

### Scenarios

- Fixing bugs: easy. Separate entry points means we can actually test things independently. Having every parameter laid out clearly makes ie easy to inspect.
- Adding new algorithms: easy. Having the functions show their explicit arguments, have explicit tests, etc. makes it easier for any new developer to add a new algorithm. 
- Updating defaults: easy--difficult, depending on implementation. Perhaps we could introduce functions for defaults?

Pros | Cons
--- | ---
Scenarios #1, #2 will be simple and straightforward | Scenario #3 may be harder to coordinate across the different interfaces
For every algorithm, there is a function in the Stan Library | For every algorithm, there is a function in the Stan Library
Testing is easy | Adding a new algorithm (say another 24) requires work
Easy to maintain | 


## Proposed Solution #2: Michael's solution

In order to reduce the difficulty in maintaining calls to every single entry point within an interface, functions in the Stan Library will accept configuration objects. The configuration objects will drive the functions. Instead of 12 total functions for HMC / NUTS, there would be 2 functions that would accept 3 config classes as arguments.

For example, the function for NUTS will look something like this:

```C++

struct nuts {
  int num_warmup = 1000;
  int num_samples = 1000;
  int num_thin = 1;
  bool save_warmup = false;
  int tree_depth = 10;
};

struct diag_e_metric {
...
};

struct windowed_adaptation {
...
};


template <class Model, class rng_t, class Metric, class Adaptation>
int nuts(Model& model,
         rng_t& base_rng,
         Eigen::VectorXd& cont_params,
         nuts& nuts,
         Metric& metric,
         Adaptation& adaptation
         interface_callbacks::interrupt::base_interrupt& interrupt,
         interface_callbacks::writer::base_writer& sample_writer,
         interface_callbacks::writer::base_writer& diagnostic_writer,
         interface_callbacks::writer::base_writer& message_writer);
```

With the current set of algorithms, we would now have 2 major entry points that have to be maintained, but the logic of creating and setting configuration objects would be difficult. For example, it would look something like this:

- CmdStan:

```C++
if (engine->value() == "nuts") {
  stan::services::nuts nuts;
  if (dynamic_cast<stan::services::int_argument*>(base->arg("max_depth"))->value())
     nuts.tree_depth = dynamic_cast<stan::services::int_argument*>(base->arg("max_depth"))->value();
  ...
}

...
if (metric->value() == "diag_e") {
  stan::serivces::diag_e_metric metric;
  if (...)
    metric.__ = dynamic_cast<stan::services::int_argument*>(base->arg("max_depth"))->value();
  ...
}

if (adapt_engaged) {
  stan::services::windowed_adaptation adaption;
  if (...)
     adapt.___ = ___;
  ...
}

return stan::services::sample::nuts(model, base_rng, cont_params,
                                    nuts, metric, adaptation
                                    interrupt, sample_writer,
                                    diagnostic_writer, info);
}
```

Note: this is much more obfuscated, but will cover all the sampling statements with a lot more care in configuration.

- RStan and PyStan look very similiar:

```C++
...
if (args.get_ctrl_sampling_algorithm() == "nuts") {
  stan::services::nuts nuts;
  if (args.get_ctrl_sampling_max_depth())
     nuts.tree_depth = args.get_ctrl_sampling_max_depth();
  ...
}

...
if (metric == DIAG_E) {
  stan::serivces::diag_e_metric metric;
  if (...)
    metric.__ = dynamic_cast<stan::services::int_argument*>(base->arg("max_depth"))->value();
  ...
}

if (args.get_ctrl_sampling_adapt_engaged() == false) {
  stan::services::windowed_adaptation adaption;
  if (...)
     adapt.___ = ___;
  ...
}

return stan::services::sample::nuts(model, base_rng, cont_params,
                                    nuts, metric, adaptation
                                    interrupt, sample_writer,
                                    diagnostic_writer, info);
}
```

This will get tricky down in the implementation between figuring out what parameters were passed through to the functions since R and Python allow optional parameters. This will, however, make it easy to specify defaults.


### Scenarios

- Fixing bugs: hard. Combined entry points means to check if there's a bug in an algorithm, we have to create configuration to call the particular algorithm. Then we have to check whether the interface is passing the correct configuration. And the code becomes more obscure.
- Adding new algorithms: easy--hard. It's easy if you know how to do it. It's hard if you're just trying to add one more algorithm. And it's hard to write explicit tests calling for that argument. 
- Updating defaults: easy. Won't even need to touch the interfaces.

Pros | Cons
--- | ---
Scenario #3 will be simple | Scenario #1 will be very difficult.
There will be less code in the interfaces due to having the fewer entry points | Scenario #2 will be difficult and the fewer entry points will result in more complex interface code
Adding a new algorithm (say another 24) will be easier | Testing is much, much harder
.      | Hard to maintain 


## Discussion

Comments by @ariddell: The title of this page should be [Cahiers de doléances](https://en.wikipedia.org/wiki/Cahiers_de_dol%C3%A9ances). The C++ code in PyStan referenced in "Overview" is borderline unmaintainable, in my opinion. Until it's maintainable and legible, it's extremely difficult to recruit new developers to PyStan. Until it's maintainable it's difficult to add new features and fix bugs. Bref, I second everything expressed on this page.

Krzysztof: I would like to get more of Daniel's perspective on the "Hard to Maintain" point about Michael's design.  My thoughts on Michael's approach is that I like it as a design for limiting the number of entry points and argument repetition across functions but as presented it could use a layer of helper functions to make it easier to call from the multiple interface languages...  Realistically I think either refactor approach would let us introduce new developers to Stan if we document thoroughly at each level (so a separate C++ API doc in the spirit of something like the [libpqxx](http://pqxx.org/devprojects/libpqxx/doc/stable/html/Tutorial/) tutorial or the [mongodb](https://github.com/mongodb/mongo-cxx-driver/wiki/Quickstart-Guide-(New-Driver)) quick start guide (but a little more thorough).
