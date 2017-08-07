General categorization of approaches to Bayesian computing with big data:
- approximate models
- approximate algorithms
- data subsetting
- faster computing for big models

Sebastian's interested in this project.

This is related to the "workflow" issue that keeps coming up.

Some things to do:

- Fast regression with point estimates and fixed priors (using rstanarm using the optimize=TRUE setting to replace bayesglm):  Currently rstanarm in optimize=TRUE setting is kinda slow because it does simulations and computes second derivatives.  I'm not quite clear on which of these things (or others) is the CPU hog, but we'll want some "light" versions that will run fast.

- GMO (are you thinking this will be fast because it can be parallelized?)

- data-parallel EP

- stochastic VB

- if it's a simple linear regression, you can use the data-parallel version in RStanArm

- CmdStan doesn't have this overhead, and also if there are a lot of parameters, won't run you out of memory storing the draws if you run in parallel

- the really big speedups on the horizon are
    - MPI for multi-core or multi-computer parallelism;  this will let us split the likelihood into bits in general, not just in the linear case supported by RStanArm
    - GPU support for matrix operations;  if we get some way to farm the data out once rather than continually transmitting it as in the Cholesky factorization prototype we have now, then we'll close the gap with somehting like Edward on very simple models that are dominate by a data matrix times parameter vector multiplication