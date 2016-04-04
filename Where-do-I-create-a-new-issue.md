# Overview

Stan development happens across multiple repositories. The diagram below shows the bigger picture. (Arrows indicate dependencies.)

          
```               
math <- language <- algorithms <- services <- pystan
                                           <- rstan   <- rstanarm
                                                      <- rethinking
                                           <- cmdstan <- statastan
                                                      <- matlabstan
                                                      <- stan.jl
```

Currently, the `stan-dev/stan` repo spans three logical layers:
- language
- algorithms
- services

Everything else has its own repository.

What we are trying to do with the services refactor is make a clean
services layer between the core interfaces (pystan, rstan, cmdstan) so that
these don't have to know anything below the services layer.   Ideally,
there wouldn't be any calls from pystan, rstan, or cmdstan other than
ones to the stan::services namespace.  services, on the other hand, is
almost certainly going to need to know about things below the algorithms
level in language and math.


# Where to add issues

Since the development of Stan happens across multiple repositories now, adding a single feature may span multiple repositories. Take a look at the diagram and add issues to each of the repositories that need to change in order to accomodate the feature request or issue.

Since that statement is a bit abstract, below are more concrete use cases.


# Use Cases

## Add a new function to the language

1. To add a new function to the Stan language, you'll add an issue to the `language` repo, which is `stan-dev/stan`.
2. If the implementation of the function doesn't already exist, you'll also add an issue to `stan-dev/math`.

## Add a new algorithm

1. To add a new algorithm, you'll add an issue to the `algorithms` repository, which is currently `stan-dev/stan`.
2. You'll also need to add an issue to `services` (currently, this doesn't require a new issue since it's also in `stan-dev/stan`)
3. You'll need to add issues to `rstan`, `pystan`, and `cmdstan` to update the interfaces to call the new algorithm.

## Make a function faster

1. Add an issue to the `math` repository. This propagates downstream.


## Request an RStan feature request

1. Add an issue to the `rstan` repository.
2. If this was for a downstream project like `rstanarm`, also add a new issue to the downstream project.
