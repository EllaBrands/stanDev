# Stan Road Map

This document is intended to serve as a broad outline of the future
direction of the Stan project as a whole.  The Stan project is
entirely open source.  The contours of the road map and progress made
along it will be driven by a wide variety of developers, advisors, and
project managers.  Stan is powered by an even broader community of
users and instructors.  Improvements in core algorithms will largely
be powered by a global network of researchers studying statistics,
algorithms, and systems.



## Algorithm Research


### Simulation-based calibration (SBC)

* methodology for large-scale, precise calibration generalizing the
  approach of Cook, Gelman and Rubin

* (interfaces) implementing methodology


### Autodiff-based variationali inference (ADVI)

Approximate Bayes with variational inference with multivariate normal
variational family (on the unconstrained scale) and a nested Monte
Carlo estimate of the gradient of an expectation.

* figure out where it works and why
    - diagnose optimization failures
    - diagnose approximation failures
    - develop adaptation and optimization algorithms

* explore alternative family parameterizations

### Gradient-based Marginal Optimization (GMO)

Used for marginal posterior modes or max marginal likelihood (MML).

*  Algorithm requires same nested Monte Carlo as ADVI
    - Find phi* = argmax_theta p(phi)
    - where p(phi) = INTEGRATE_Theta p(phi, theta) d.theta.


### Expectation propgation

Used for data-parallel approximate Bayes.  Still largely a work in
progress.

* Develop algorithm
    - choose version to use, tuning params, etc.
    - implement serial version in C++ using Stan's log density and gradient
    - simulate parallel version in C++
    - implement parallel version with MPI (?)


### Riemannian Hamiltonian Monte Carlo (RHMC)

* depends on:  testing higher-order autodiff
* integrate into build process
* integrate into interfaces

### Core Algorithms

* More flexible and continuous adaptation for HMC/NUTS

* More flexible output control through interfaces
    - particularly silent mode through RStan


## Stan Language

### 64-bit integers

Ben Goodrich is a long way along scripting this.  Turns out 32 bits
really are limitations for integers and people want to use integers
north of 2e+9.


### Data types

This will include language components for constructing, accessing
components, and assignment.  It will also include basic I/O for
constraining, unconstraining, and these will need to be propagated
through the services to the interfaces, which will themselves need to
deal with formats for the new data types in their native languages.

* tuples
* ragged arrays
* sparse matrices and vectors
* functions
* boolean (allow easier vectorized indexing); subtype of int

### Expressions

* function closures (lambdas with binding)
* general function application

### Covariance

Extend legal assignments and function calls so that Stan's type system
is fully covariant.  Syntactically, if A is a subtype of B, it means
an expression of type A can be used wherever an expression of type B
is required.  Semantically, the set of denotations of expressions of
type A will be a subset of those of type B.  Covariance is defined
inductively on the basis of primitive types, which for Stan is only a
single relation.

* primitive
    - arithmetic:  int is a subtype of real

To be covariant requires closure under inductively defined types, so
that if A' is a subtype of A, then

* containers
    - (ragged) arrays:  A'[] is a subtype of A[]
    - (sparse) matrices: only real matrices, so not relevant

* functions:
    - domain: A -> B is a subtype of A' -> B
    - range:  B -> A' is a subtype of B -> A

### I/O

* move to standard, binary I/O format for file-based and network
  I/O
    - protocol buffers
* support streaming, peekable, output
    - GUI progress monitor
*

### Language documentation

* Move manual to bookdown format to generate HTML and pdf

* Formal semantics with type theory and execution


### Compound function acceleration

* GLM functions


### More general vectorization

* binary functioins and beyond
* multivariate functions
    - matrix functions
    - lpdf and lpmf
* vectorize comparison
    - use result for indexing

### Model combinators

More of a research problem in how to combine models.  This can't be
done through functions because there is no way to introduce
parameters, data, etc.


### Standoff abstract syntax tree

This will allow for much easier compiler optimizations and code
generation for different targets as well as providing an easier target
for alternative interfaces.

### Compilation efficiency

Right now, the time to compile a Stan model is an enormous bottleneck
for model exploration.  Our goal is to reduce that substantially;  we
don't really know how much improvement is possible

* Precompile as much of the math and model library as possible
    - all matrix operations can be
* Precompile generic instantiations of distribution library, e.g.,
    - normal(T1* y, int size_y,
             T2* mu, int size_mu,
	     T3* sigma, int size_sigma)
      for T1, T2, T3 in { var, double }
    - delegate all the current templated calls

### Plug-in language extensions

Allow users to define new functions in C++ and easily include them in
Stan.

* this will only cover use cases in Torsten (Metrum's pharma framework for
  RStan) where they define new integrators when we have functional types

### Flexible derivatives for subsets of parameters

We need this for a lot of the algorithm development where some
parameters are kept fixed and derivatives calculated for others.
Requires a way to specify from the outside or inside the language.



## Diagnostic error messages

We need to improve error message understandability to users.  Messages
that seem clear to developers are often perplexing to users, who are
more often domain experts than seasoned programmers or computational
statisticians.

* Our biggest push is going to be to remove extraneous false positive
  warnings
    - compiler warnings from C++
    - warnings from the Stan language parser

* Error messages from the language with clearer indications of
  arguments, indexing position, etc.

* Catching errors in configuration, calling pattern, etc., as early as
  possible in an algorithm, interface, or Stan program (e.g., check
  all the arguments are in place before compling a Stan program in an
  RStan call to stan())

* Error messages from algorithms indicating failures need to have
  clearer descriptions and ideally next actions for users


## Robustness


## Multi-Process Parallelization

This provides massive scalability and efficiency gains for likelihood
functions, which are typically naturally parallelizable in way that
leads to low communication overhead for gradients and values.  It
works over mulitple machines on a network or multiple cores on a
single machine.  It can also speedup single-core computations by
reducing memory overhead with nested gradient reductions in the
forward pass of reverse-mode autodiff.

* Implement generic map function using MPI
    - simple map/reduce abstraction layer over MPI
    - push data to compute nodes for static data arguments
    - nested gradient calculations for reduction into expression graph


## GPU Parallelization

GPUs provide up to an order of magnitude speedup using inexpensive
hardware for double-based calculations involving large, regular data
structures, such as matrices.

* Implement matrix operations on GPU with analytic gradients
* Initially focus on quadratic data, cubic computation algorithms
  that are widely used in multivariate models (such as Gaussian
  processes)
    - Cholesky factorization
    - matrix multiply
    - log determinant



## Services Framework

* Extend to new I/O frameworks

* Flexible default configuration for services
    - function argument builder pattern, e.g.
      adaptive_euclidean_nuts.stepsize(0.1).adapt_delta(0.99).run();
    - ideally the bit before .run() would be reusable
    - builder (lower case) pattern for function

* Allow Stan to be easily embedded in server-side applications.

* Create AWS(?) version as example of Stan-as-a-service


## Autodiff Testing Framework

One of the main bottlenecks to developing new functions in the math
library with autodiff is the challenge of testing all the
instantiations of a function.  This can largely be automated with a
general enough testing framework.  It's been started and applied to
scalar arguments in test/unit/mix for all the built-in operators.

* extend to multivariate input types (matrix, vector, array)

* extend to integer types (arguments only---return doesn't need
  autodiff)

* extend to vectorized functions
    - reductions like distributions
    - pointwise vectorizations like the math functions

## Operations

The main goal here is to make it easier to develop for Stan.  This
encompasses our continuous integration and builds for devlopers.

* better error messages from Jenkins (this may be solved)

* alternative to Travis with fewer timeouts

* cluster-based and AWS testing to remove bottlenecks

* maintaining robust build scripts as systems and compilers evolve and
  Stan's compiler dependencies grow (e.g., CVODES binary)


## Installation

* checklisted installation procedures by platform
* pre-built binary installers


## Applications

Push into new application areas.  This will largely be driven by the
develoment of tutorial materials often coupled with new language
features or algorithms.


## Teaching, Meetups, and Tutorials

* probability theory and statistics
* applied statistical modeling
* programming (Stan, R/Python/...)
* computational statistics
* Stan, RStan, RStanArm, PyStan, ...


## Discourse Forums

http://discourse.mc-stan.org

The current admins (02/2018) are:
- Daniel Lee
- Jonah Gabry
- Allen Riddell
- Sean Talts

The mods are:
- Bob Carpenter

Our plan is limited to total 5 mods or admins at a time. We switch admins from time to time.

Do we need a more online alternative such as Slack? Sean set one up here: mc-stan.slack.com


## Stan Conferences (StanCon)


## Web Site


## Governance

We need governance for a range of issues

### Who is a developer?

Basically anyone who submits a substantial pull request to one of our
official repositories with documentation and tests.  In special cases,
we've added advisors who work closely with us on algorithms to the
list of developers.

### Software governance

The current situation is that we try to work by consensus, deferring
to experts in whatever is being done.  To break deadlocks, a single
person has final decision making power for each module of code.   The
big question facing us is whether to move to a more purely democratic
form of governance putting everything to a vote among the developers
at large and trusting those without knowledge of an issue to abstain.

#### Persons in charge of Stan modules

Other than the Stan repo, which contains the language, algorithmns, and
services, all the other modules reside in their own GitHub repositories.

* Math library:  Daniel Lee
* Stan language:  Bob Carpenter
    - Stan manual:  Bob Carpenter
* Stan algorithms:  Michael Betancourt
* Stan services:  Daniel Lee

* CmdStan: Michael Betancourt
* RStan:  Ben Goodrich
    - RStanArm:  Ben Goodrich
    - ShinyStan:  Jonah Gabry
    - BayesPlot: Jonah Gabry
    - loo: Jonah Gabry
* PyStan:  Allen Riddell

* MatlabStan: Brian Lau
* Stan.jl (Julia): Robert Goedman
* StataStan:  Robert L. Grant
* MathematicaStan: Vincent Picaud

* Web site: Breck Baldwin

## Licensing

* Have to decide how to license/enforce trademarks on Stan and the logo

* PyStan3 will move to the ISC license (simplified MIT without
  copyleft)


# Stan 3

This is the big one.  When we've gathered all the interface changes we
think we want to keep for a longer run and have run a while with
existing functionality deprecated, we will release Stan 3 and remove
all the old deprecated functionality (thus breaking backward
compatibility).

This is unlikely to happen in 2018 and may not even happen in 2019.

The big struggle is going to be the increasing amount of material
being developed for our current interfaces.  Hopefully the nagging
about deprecated features will have already been enough to get them to
update, but it's a pain to keep up with this kind of thing, so it's
going to be painful no matter what we do.
