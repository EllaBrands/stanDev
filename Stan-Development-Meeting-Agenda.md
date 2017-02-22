### Roundtable
_Short (less than 1 minute) description of work in the past week._

### Unresolved Topics
_Address any open topics from the past meeting._

### New Topics
_New topics that should be addressed immediately or by the next
meeting._

* __Developer name.  Short description.  Desired resolution.__

- Sebastian. Warning from NUTS for too small stepsize? Currently we allow the stepsize to be reduced without any bound down to the machine precision. (i) Would a minimal stepsize make sense? (ii) We should fire a warning or even terminate once the stepsize is at or too close to the machine precision. Decision on ii.

### Open Discussion Topics
_Any topics that do not need to be addressed in the short term,
including speculation and brainstorming._

* __Developer name.  Short description.__

Sebastian. adapt_delta targets for numerically imprecise gradients? Is research done on this topic?

Andrew.  Nonidentifiability in symmetric probabilistic matrix factorization model.

Andrew.  Priors as part of parameter specification in Stan language?

Charles. For algebraic solver: evaluating the Jacobian of the solver with respect to the parameters. I'm a little confused about which variables are fixed and which ones vary.

Charles. Passing tuning parameters for ODEs in higher-order function. Function gets properly parsed, math unit tests work, but final implementation doesn't.

