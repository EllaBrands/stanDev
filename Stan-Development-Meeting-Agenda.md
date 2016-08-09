### Roundtable
_Short (less than 1 minute) description of work in the past week._

### Unresolved Topics
_Address any open topics from the past meeting._

* Ben Should user-defined functions be part of the class? Yes
* Ben https://github.com/stan-dev/stan/wiki/Functionals-spec Discuss
* Ben One day webinar with the FDA in August. Propose date
* Carpenter.  How should RNGs in transformed data work w.r.t. chains and seeds?  Decision.
* Carpenter.  Do we pass chain IDs into transformed data and other blocks?  Don't care, but we should decide.
* Carpenter.  Are we C++ ready and if so, will anyone get left behind by the switch (we don't want multiple maintenance paths)?  I'd be very happy if the answer is yes and nobody's left behind.

### New Topics
_New topics that should be addressed immediately or by the next
meeting._

* __Developer name.  Short description.  Desired resolution.__

### Open Discussion Topics
_Any topics that do not need to be addressed in the short term,
including speculation and brainstorming._

* __Developer name.  Short description.__

* Sebastian. Do we need to have a prim version of integrate_ode_rk45? I would like to make coupled_ode_system inherit from ode_system. The ode_system includes autodiff code and uses Eigen. The consequence is that all ODE code ends up in rev/mat. If ode_system is the base of coupled_ode_system, then it brings the Jacobian wrt to states with it. This is required for stiff integration (not required for non-stiff).

* Sebastian. Parallelization with OpenMP. A prototype works - ways to roll this into Stan?