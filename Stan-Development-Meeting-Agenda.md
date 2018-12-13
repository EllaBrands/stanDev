### Roundtable
_Short (less than 1 minute) description of work in the past week._

### Unresolved Topics
_Address any open topics from the past meeting._

### New Topics
_New topics that should be addressed immediately or by the next
meeting._

* __Date added. Developer name.  Short description.  Desired resolution.__

13 Dec.  Andrew and Aki.  Stan output.

13 Dec.  Andrew etc.  ADVI in R.

6 Dec. Matthijs. Static analyses for Stan. Can we exploit conditional independence properties in HMC? Conjugate priors?

6 Dec. Matthijs. Talk about Stan at POPL. Any suggestions on what to showcase?

6 Dec. Matthijs. Belief propagation for discrete parameters in Stan. Does anyone see problems with this?

28 Nov.  StanCon2019 is in Cambridge: Some general questions, request for feedback.

14th Nov. Charles. Do we want a Newton solver in Stan. When would it work better in than Powell's method.

14th Nov. Charles. Do we want an optimization function (i.e. max objective function).

15 Nov. Matthijs: Functions reference document is broken.

All below old and done?

1st Nov. Sebastian. Can we have an optional `coupled_ode_system` in stan-math which enables an easy way to provide analytic Jacobians for ODE solving (leads to >2x speedups for cases I have seen)? We could allow for this using compile time flags, for example.

1st Nov. Sebastian. Does it make sense to have ODE integrators provide the solutions of the states with higher relative accuracy vs the gradients? Does it buy sampling performance something (knowing lp more precisely vs the gradients is worthwhile, I think)? Are we up to yet another ODE call?

### Open Discussion Topics

_Any topics that do not need to be addressed in the short term,
including speculation and brainstorming._

* __Date added. Developer name.  Short description.__
