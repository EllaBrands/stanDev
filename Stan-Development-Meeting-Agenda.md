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
* Carpenter.  How should RNGs in transformed data work w.r.t. chains and seeds?  Decision.
* Carpenter.  Do we pass chain IDs into transformed data and other blocks?  Don't care, but we should decide.
* Carpenter.  Are we C++ ready and if so, will anyone get left behind by the switch (we don't want multiple maintenance paths)?  I'd be very happy if the answer is yes and nobody's left behind.

### Open Discussion Topics
_Any topics that do not need to be addressed in the short term,
including speculation and brainstorming._

* __Developer name.  Short description.__

* Sebastian. chain ID / chain specific data: Either by chain ID or by multiple input data files for cmdstan.
* Sebastian. Hypergeometrics 2F1: If we don't have a definition (confirm), then I may just drop the issue
* Sebastian. Nested includes?
