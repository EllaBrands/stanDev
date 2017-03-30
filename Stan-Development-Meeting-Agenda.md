### Roundtable
_Short (less than 1 minute) description of work in the past week._

### Unresolved Topics
_Address any open topics from the past meeting._

### New Topics
_New topics that should be addressed immediately or by the next
meeting._

* __Developer name.  Short description.  Desired resolution.__

Betancourt.  StanCon 2018 go ahead.  Yay or nay.

Betancourt.  Any interest in a stats-is-important paper for Nautilus?  http://nautil.us. Whether or not to reach out to contact.

Betancourt.  Shall we transition the Users' List to Discourse?  What would be first steps?

Sebastian. Non-stiff ODE speedup pull-request (https://github.com/stan-dev/math/pull/513). Short description of changes. Reviewer candidates.

### Open Discussion Topics
_Any topics that do not need to be addressed in the short term,
including speculation and brainstorming._

* __Developer name.  Short description.__

Sebastian.  Parallelization. Do we want to go into the direction of https://github.com/stan-dev/stan/wiki/Parallelism-in-Stan ? Essentially the idea is to introduce specialized "wahtever_parallel" functions which use OpenMP. Do we want something else?

Sebastian.  Parallel ODE: The prototype is fully done, i.e. I have created a `stan_ode_model` R function. Do we want to make this available in some form and if so how? An extra rstanode repo / a branch in rstan repo?

Betancourt.  How to move forward on changing CmdStan output?