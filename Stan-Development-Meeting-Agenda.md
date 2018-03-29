### Roundtable
_Short (less than 1 minute) description of work in the past week._


### Unresolved Topics
_Address any open topics from the past meeting._

### New Topics
_New topics that should be addressed immediately or by the next
meeting._

* __Developer name.  Short description.  Desired resolution.__

Sean Talts. Different goals and use-cases for performance, regression, and end-to-end numerical accuracy testing.

Sebastian Weber. Do we want C++11 map_rect_async asap? Concerns? Is Apple clang a must have? Should this be optional (compiler defs) and can we configure threads by STAN_THREADS environment variable?

Sebastian Weber. Parallelism beyond MPI in stan-math. Do we want openmp? Or better go with C++11/17 facilities (TS parallelism likely in 17 or use of [parallel libstdc++](https://gcc.gnu.org/onlinedocs/libstdc++/manual/parallel_mode_design.html)). Plan for moving forward would be great (immediate decision or ongoing discussion).

### Open Discussion Topics

_Any topics that do not need to be addressed in the short term,
including speculation and brainstorming._

* __Developer name.  Short description.__

Andrew.  Various user issues
- Running Stan for fixed amount of time instead of fixed number of iterations
- other issues