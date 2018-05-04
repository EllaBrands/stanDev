Introduction:
- Big Data needs Big Model
- We're addressing the following problem:  We would like to fit a fully specified Bayesian model M on data y, thus obtain draws theta from p(theta|y,M).  But when we run in Stan it's too slow; we do not reach any approx convergence before it's time to get the damn results already.
- In background:  Yes, this is all part of a "workflow" with other models being considered.  Indeed, one reason for wanting to fit faster is to be able to try out more models.  Also the data could be part of a workflow too in that we may be gathering more, although that is not the main concern of this paper.
- What do people do?  Run Stan and stop early; run other programs that fit particular models that aren't quite what we would like to fit; fit to a subset of data; do some approximate Bayes algorithm.  Concern that any or all of these approaches could be wrong in any particular case.
- Summary:  Our goal is to approximate p(theta|y,M) and get some sense of the accuracy of the approximation.

General categorization of approaches to Bayesian computing with big data:
- approximate models
- approximate algorithms
- data subsetting
- faster general computing that allows one to fit many big models (for example, GPUs for matrix algebra)
- faster computing for specific models (for example, integrating out the linear parameters in a hierarchical linear regression)

Sebastian's interested in this project.

Dan Simpson points the relevance of asymptotic arguments.

Some things to do:

- Fast regression with point estimates and fixed priors (using rstanarm using the optimize=TRUE setting to replace bayesglm):  Currently rstanarm in optimize=TRUE setting is kinda slow because it does simulations and computes second derivatives.  I'm not quite clear on which of these things (or others) is the CPU hog, but we'll want some "light" versions that will run fast.

- GMO

- data-parallel EP

- stochastic VB

- if it's a simple linear regression, you can use the data-parallel version in RStanArm

- CmdStan doesn't have this overhead, and also if there are a lot of parameters, won't run you out of memory storing the draws if you run in parallel

- the really big speedups on the horizon are
    - MPI for multi-core or multi-computer parallelism;  this will let us split the likelihood into bits in general, not just in the linear case supported by RStanArm
    - GPU support for matrix operations;  if we get some way to farm the data out once rather than continually transmitting it as in the Cholesky factorization prototype we have now, then we'll close the gap with somehting like Edward on very simple models that are dominate by a data matrix times parameter vector multiplication

- Example.

  - Data on grades indexed by students and teachers:  Each student has several grades from each teacher in each year.  Use item-response model:  grade = mu + student_ability - teacher_difficulty + error.  Students and teachers each have aging curves so that ability and difficulty vary over time.  Can fit big model in Stan but it's slow; that's ok.  Now want to update each week in new school year given new data.  Particularly challenging cases are teachers and students who previously were in lower school systems.  Approaches to updating the posterior distribution with new week's information:
    - Importance weighting.  Should be fast but could have difficulty when new parameters are being introduced.
    - Approx the posterior dist of teacher parameters and hyperparameters with multivariate normal, then use this as a prior to fit full data independently for each student.  Similarly fit model independently for each teacher, using previous posterior of teacher parameters and hyperparameters as an approximate prior.

  - In any case, the plan would be to update the entire model off-line every couple weeks or whatever.  We're not planning to do full "particle filtering"; we just want to do these quick updates.
Key feature of problem that we want to leverage is that, conditional on hyperparameters and teacher parameters, the post dist is indep in the student parameters.  Similarly, conditional on hyperparameters and student parameters, the post dist is indep in the teacher parameters.  We can use this:
    - In imp weighting, we can independently update for each student and each teacher (ignoring, for each student, the updated info on each teacher, and vice versa).
    - In the independent posterior distributions, we can fit each student (and, separately, each teacher) in parallel.