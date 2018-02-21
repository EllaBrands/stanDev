# Stan Road Map

This document is a broad plan for the future direction of the
*software products* of the Stan project.  Stan is very much a group
effort and we welcome contributions at every stage of software
develoment: algorithms, API design, coding, documentation, and testing.
If you're interested in getting involved in development, here's the [Stan developers home page](http://mc-stan.org/developers/).

#### Open source licensing

The code underlying Stan is 100% open source (core is BSD, interfaces
copyleft where required).

#### Software governance

The Stan project is organized around functional modules; this roadmap
follows the same module structure as the software.

Stan's software is managed largely by consensus of the developers.
Becoming a developer is as easy as having a substantial pull request
with tests and documentation merged into develop; merge and review
priveleges are granted after several months of ongoing contributions.
A single developer has final authority in each repository for
situations in which consensus cannot be reached (such intervention has
only been necessary a couple times over six years).

#### Feature process

1. Features begin as discussions among Stan developers and users.  The
best place to do this is the Stan Discourse forums.
2.  The broad feature desiderata are converted to functional
specifications, which convey what the feature will look like in the
final product from a client perspective.  For large features, this is
typically done through a GitHub Wiki page.
3.  The functional specification is broken down into a technical
specification laying out how it may be implemented.  This is typically
done on the GitHub issue tracker.
4.  One or more developers implements the feature (code, tests, and
documentation) in a branch from one or more of the Stan GitHub
repositories.
5.  The code goes through continuous integration testing, then one or
more core developers review the code through GitHub.  The reviewer may
suggest or require improvements, sending the process back to step 4
(or sometimes earlier).  When the reviewer is satisfied, the code is
merged into the development branch of the relevant repository, where
it will be included in the next stable release.

We strongly encourage the community to get involved in every stage of this process.

#### Stan 3

After we've gathered all the interface changes we think we want to keep
for a longer run and have run a while with existing functionality
deprecated, we will release Stan 3 and remove all the old deprecated
functionality.  This will break backward compatibility.  It is very unlikely to happen in 2018.

PyStan 3 will happen earlier to enable ISC (non-copyleft) licensing;  it won't coincide with any change in the language.



# Stan Modules

This section lists the Stan modules and developer in charge of them as
well as upstream dependencies (modules on which the stated modules depends)
and downstream dependencies (modules which depend on the stated package).

#### Core C++ Modules

| Module | Repository | Manager | Upstream | Downstream |
| ------ | ---------- | ------- | -------- | ---------- |
| Math Library  | `stan-dev/math` | Daniel Lee | | language |
| Language | `stan-dev/stan` | Bob Carpenter | math | algorithms |
| Algorithms | `stan-dev/stan` | Michael Betancourt | math | language |
| Services | `stan-dev/stan` | Daniel Lee | algorithms, language | interfaces |

#### Interfaces

| Module | Repository | Manager | Upstream | Downstream |
| ------ | ---------- | ------- | -------- | ---------- |
| CmdStan | `stan-dev/cmdstan` | Michael Betancourt | services | |
| RStan | `stan-dev/rstan` | Ben Goodrich | services, bayesplot | |
| PyStan | `stan-dev/pystan` | Allen Riddell | services | |
| Stan.jl | `goedman/Stan.jl` | Robert Goedman | services | |
| MatlabStan | `brian-lau/MatlabStan` | Brian Lau | services | |
| StataStan | `stan-dev/statastan` | Robert L. Grant | services | |
| MathematicaStan | `stan-dev/MathematicaStan` | Vincent Picaud | services | |

#### Higher-Level Interfaces

| Module | Repository | Manager | Upstream | Downstream |
| ------ | ---------- | ------- | -------- | ---------- |
| RStanArm | `stan-dev/rstanarm` | Ben Goodrich | rstan, rstantools, loo, bayesplot, shinystan | |

#### Posterior Analysis, Visualization, Packaging

| Module | Repository | Manager | Upstream | Downstream |
| ------ | ---------- | ------- | -------- | ---------- |
| loo | `stan-dev/loo` | Jonah Gabry | | rstanarm |
| bayesplot | `stan-dev/bayesplot` | Jonah Gabry | | rstan, rstanarm |
| shinystan | `stan-dev/shinystan` | Jonah Gabry | bayesplot | |
| rstantools | `stan-dev/rstantools` | Jonah Gabry | rstanarm | |

#### Web site

| Module | Repository | Manager | Upstream | Downstream |
| ------ | ---------- | ------- | -------- | ---------- |
| web site | `stan-dev/stan-dev.github.io` | Breck Baldwin | | |

# Math Library Roadmap
* *Repository*: `stan-dev/math`
* *Manager*: Daniel Lee

#### MPI parallelism (multi-core, cluster)
* Message passing interface (MPI) implementation
* Push data to processors; communicate parameters, return values and
  gradients
* Map/reduce abstraction on top of MPI for autodiff
* Expose rectangular dense map function to language

#### GPU parallelism
* Underlying matrix opertion implementations
* Push data to GPUs
* Initially target quadratic data, cubic operation functions: Cholesky decomposition, log determinant, matrix multiply

#### OpenMP parallelism (multiple threads)
* within Eigen for some matrix operations
* parallelize double calculations in distribution functions

#### Autodiff testing framework
* document what we currently have
* extend to multivariate input types (matrix, vector, array)
* extend to integer types (arguments only---return doesn't need
  autodiff)
* extend to vectorized functions including reductions like distributions and pointwise vectorizations like the math functions

#### C++11 upgrades
* simplify code with C++11 `auto`, foreach, lambdas, etc.

#### Efficiency upgrades
* reduce copying and support Eigen expression templates with more generic arguments

#### Metaprogram unification
* remove redundant metaprograms
* extend boundary conditions of existing programs

#### Vectorization framework
* framework to allow easy vectorization beyond unary scalars

#### Sparse matrix arithmetic
* basic implementations on top of Eigen
* analytic derivatives

#### Internals documentation
* explain code organization
* explain metaprograms and utility classes
* examples of how to implement and test different kinds of functions

#### Remove scal/array/matrix distinction
* Collect `scal`, `arr`, `mat` into single files under top-level organization

#### Performance monitoring and testing
* Add a way to measure speed between branches

# Language Roadmap
* *Repository*: `stan-dev/stan`
* *Manager*: Bob Carpenter

#### Multi-core parallelism
* Generic `map()` function which may be configured to use MPI with
  local data

#### Data types and operations
* *Simply typed functions*: application, abstraction with binding
   (i.e., lambdas with closures)
* *64-bit integers*:  replace current 32-bit implementation
* *Tuples*: statically typed sequences of heterogeneous types
* *Ragged arrays*: contain elements of the same shape, but varying sizes
* *Sparse matrices, vectors*: plus arithmtic, solvers
* *Boolean*: to allow easy selection; subtype of `int`

#### Calculus functions
* Differential algebraic equation (DAE) solver
* Partial differential equation (PDE) solvers
* Definite integrators, bounded and unbounded

#### General functions
* Compound generalized linear models (GLM)
* approximate GLM (a la INLA)
* Hidden Markov models (HMM)
* Gaussian process (GP) covariance functions
* Vectorization including n-ary functions, multivariate functions,
  multivariate distributions, and comparison operators

#### Documentation
* Break current manual into user's guide, language spec, and
  supporting materials
* Move manuals to HTML format with bookdown (R markdown)
* Produce formal syntax and semantics
* Parser, abstract syntax tree, generator code (dev facing)

#### Covariance
Make Stan fully covariant so any function or assignment to a type may
be used with a subtype.
* every type is a subtype of itself
* `int` is a subtype of `real`
* `A[]` is a subtype of `B[]` if `A` is a subtype of `B`
* `A -> B` is a subtype of `C -> D` if `C` is a subtype of `A` and `B`
  is a subtype of `D`

#### User-defined functions with analytic gradients
* Allow users to specify analytic gradients for functions
* Test framework for correctness of user's definition

#### Error diagnostics
* Improve user interpretability of error messages with suggestions for
  remediation
* Vectorize error handling in math lib to include positions everywhere
* Pedantic mode/lint system to take over from existing heuristics
  (eliminating parser false positive warnings)
* Remove false positive compiler warnings

#### Compilation efficiency
Most of this speedup will come through algorithms and the math library.

#### Model combinators
* Combine models consisting of parameters, data, and log density
contributions.

#### Plug-in extensions
Allow users to define new functions in C++ and easily include them in
Stan; this is much more powerful with function types.

#### Flexible derivatives for subsets of parameters
Allow a subset of parameters to be fixed and derivatives only
calculated for remainder.

#### Standoff abstract syntax tree (dev facing)
* Simple s-expressions rather than variant types to allow easy
  generation and intermediate transformation
* Pretty printer from abstract syntax tree

#### Model class (dev facing)
* properly modularized abstract base class to enable easy
  implementation by clients
* constant correctness
* flexible, structured access to declared variable declarations, data
  values, and transforms

#### Stan 3 language
* blockless model a la SlicStan with full functions
* target for StataStan or a new embedded PyStan
* rethink Spirit Qi parser choice; target standoff AST


# Algorithms Roadmap
* *Repository*: `stan-dev/stan`
* *Manager*: Michael Betancourt

#### Smooth warmup
* instead of blocked adaptation, add rolling average that adapts smoothly
* challenge will be getting step size adapted at the same time

#### Timed and sample size budgeted processing
* set a time budget, and try to generate largest n_eff in that time
* set an n_eff budget, and run only long enough to get there
* start with better estimated finish times

#### Online progress monitor (GUI)
* online monitoring of means, variances, R-hat with Welford's algorithm
* report summaries in output

#### Simulation-based calibration (SBC)
* methodology for large-scale, precise calibration generalizing the
  approach of Cook, Gelman and Rubin
* implementing necessary services for methodology

#### Autodiff-based variational inference (ADVI)
Approximate Bayes with variational inference with multivariate normal
variational family (on the unconstrained scale) and a nested Monte
Carlo estimate of the gradient of an expectation.
* diagnose optimization failures
* diagnose approximation failures
* develop adaptation and optimization algorithms
* explore alternative family parameterizations

#### Gradient-based Marginal Optimization (GMO)
Used for marginal posterior modes or max marginal likelihood (MML).
* develop algorithm
* validate vs. lme4
* optimize implementation

#### Expectation propgation
Used for data-parallel approximate Bayes.
* choose version to use, tuning params, etc.
* implement serial version in C++ using Stan's log density and gradient
* simulate parallel version in C++
* implement parallel version with MPI (?)

#### Riemannian Hamiltonian Monte Carlo (RHMC)
* depends on:  testing higher-order autodiff
* integrate into build process

#### Core Algorithms
* More flexible and continuous adaptation for HMC/NUTS
* More flexible output control through interfaces
    - silent mode through RStan



# Services Roadmap
* *Repository*: `stan-dev/stan`
* *Manager*: Daniel Lee

#### Configuration builder for service functions
* Employ builder pattern with defaults to allow, e.g.,
```
adaptive_euclidean_nuts.stepsize(0.1).adapt_delta(0.99).run();
```

#### Server-side embedding
Allow Stan to be easily embedded in server-side applications.

#### I/O

* Introduce protocol buffer binary I/O format for streaming file-based
  and network I/O



# CmdStan Roadmap
* *Repository*: `stan-dev/cmdstan`
* *Manager*: Michael Betancourt



# RStan Roadmap
* *Repository*: `stan-dev/rstan`
* *Manager*: Ben Goodrich



# PyStan Roadmap
* *Repository*: `stan-dev/pystan`
* *Manager*: Allen Riddell

#### PyStan 3

* ISC license (like MIT or BSD license)


# Web Site Roadmap
* *Repository*: `stan-dev/stan-dev.github.io`
* *Manager*: Breck Baldwin


# Miscellaneous

I'm including things that don't fit into any given module here.

#### Installers and installation procedures
* Pre-built binary installers for C++ components
* Checklisted installation procedures by platform

#### Applications
* New application areas exemplified with case studies or manual chapters

#### Forums
* Dev team and user chat (some open source alternative to Slack, preferably)

#### Wikis
* Remove irrlevant wiki pages
* Consolidate and organize wikis for easy access

#### Teaching, tutorials, and meetups
* Intro probability theory and stats to support applied Bayesian stats
* Applied statistical modeling by model type and application area
* Programming basics (R, Python, etc.) and Stan
* Computational statistics (MCMC, etc.)
* Specific interfaces

#### Conferences
* Process for selecting locations and local organizers
* Schedule

#### Governance
* Who is a developer?
* Who appoints/deposes managers of repositories?

#### Operations (dev facing)
* better error messages from Jenkins (this may be solved)
* alternative to Travis with fewer timeouts
* cluster-based and AWS testing to remove bottlenecks
* maintaining robust build scripts as systems and compilers evolve and
  Stan's compiler dependencies grow (e.g., CVODES binary)

#### Performance and accuracy regression tests