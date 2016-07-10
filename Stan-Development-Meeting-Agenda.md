### Roundtable
_Short (less than 1 minute) description of work in the past week._

### Unresolved Topics
_Address any open topics from the past meeting._

### New Topics
_New topics that should be addressed immediately or by the next
meeting._

* __Developer name.  Short description.  Desired resolution.__

* Ben Should user-defined functions be part of the class? Yes
* Ben https://github.com/stan-dev/stan/wiki/Functionals-spec Discuss
* Ben One day webinar with the FDA in August. Propose date

### Open Discussion Topics
_Any topics that do not need to be addressed in the short term,
including speculation and brainstorming._

* __Developer name.  Short description.__

* Sebastian. odeSD inclusion?
    * Pro: Faster than CVODES for large & sparse ODE systems (systems biology, spatial SIR); I have seen 7x speed increase over CVODES
    * Pro: C++ header only solver
    * Pro: stiff solver
    * Con: LGPL
    * Con: Requires ODE RHS f AND ODE RHS f' => easy to write down, but Stan language heavy
  
