### Roundtable
_Short (less than 1 minute) description of work in the past week._

### Unresolved Topics
_Address any open topics from the past meeting._

### New Topics
_New topics that should be addressed immediately or by the next
meeting._

* __Developer name.  Short description.  Desired resolution.__

Betancourt.  What's holding up the new Cholesky branch?  Merge or not to merge, otherwise what needs to be done.

Margossian. root finder: I realize we were going with Eigen's dogleg method, but we should consider KINSOL's solver, which will be easier to parallelize. 

Gabry. Need to decide on rstanarm default behavior with priors before Andrew's book. We have a meeting to discuss rstanarm issues Friday AM but it would be good to get everyone's opinion today and then a subset of us can discuss in more detail Friday. 

Sebastian. Parallel ODEs in Stan using OpenMP (linear speedup with # CPUs!): It works by hijacking the ODE facilities. Do we want this in Stan to be available to all users? If yes, then: Need for ODE system refactor (stiff+non-stiff)? Who wants to join? More work needed: Analytic Jacobians of ODE rhs (optionally with code-generation); ragged arrays.

### Open Discussion Topics
_Any topics that do not need to be addressed in the short term,
including speculation and brainstorming._

* __Developer name.  Short description.__

Andrew.  Priors as part of parameter specification in Stan language?

Andrew.  Nonidentifiability in symmetric probabilistic matrix factorization model.

Betancourt.  Gaussian process covariance function implementations -- compute as intermediary object or just produce the needed probabilistic objects (densities or Cholesky factors)? 