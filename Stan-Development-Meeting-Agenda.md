### Roundtable
_Short (less than 1 minute) description of work in the past week._

### Unresolved Topics
_Address any open topics from the past meeting._

Daniel and Bob.  Should we have some exceptions terminate all algorithms (e.g.,
`std::out_of_range`) and some that don't terminate (e.g., `std::domain_error`)?
Decision.

Sebastian.  Should we support Intel's math lib (MKL) with specialized builds?
Decision.

Rob Trangucci. Use cases for lower-triangular and symmetric matrix types in Stan Math.

### New Topics
_New topics that should be addressed immediately or by the next
meeting._

* __Developer name.  Short description.  Desired resolution.__

Bob Carpenter.  Overflow exception vs. -infinity return for log(0), log1p(-1), log1m(1), and beta_lccdf(1 | a, b), beta_lcdf(0 | a, b).  Decision.

### Open Discussion Topics
_Any topics that do not need to be addressed in the short term,
including speculation and brainstorming._

* __Developer name.  Short description.__