### Roundtable
_Short (less than 1 minute) description of work in the past week._

### Unresolved Topics
_Address any open topics from the past meeting._

* Is there going to be a 2.10 release soon?

### New Topics
_New topics that should be addressed immediately or by the next
meeting._

* __Developer name.  Short description.  Desired resolution.__

* Daniel. Stan v2.10. When do we code freeze?

* Bob. Error handling for functions.  Decide whether to (a) follow the C++ std library in error handling (e.g., log(-1) returns NaN but doesn't raise an exception), or (b) throw when we detect an unrecoverable error?  

### Open Discussion Topics
_Any topics that do not need to be addressed in the short term,
including speculation and brainstorming._

* __Developer name.  Short description.__

 * Betancourt.  What should the absolute accuracy goals be for the ODE integrators?
 * Ben / Allen. Should we support the Feather format?
 * Sebastian. Introduction round; Discussion on ode tolerances; merge of robust integrator pull into 2.10?
 * Aki: test models framework?