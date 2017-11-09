### Roundtable
_Short (less than 1 minute) description of work in the past week._


### Unresolved Topics
_Address any open topics from the past meeting._

### New Topics
_New topics that should be addressed immediately or by the next
meeting._

* __Developer name.  Short description.  Desired resolution.__

* Sebastian+Sean. We would like some input on the MPI implementation we are aiming at, i.e. should we aim for a fully generic `reduce` or a little more specific implementation. A more generic solution comes at the price of a greater complexity, longer development time and possibly some performance loss, but has the potential upside to be easier to adapt for future parallel functions. The more specific implementation would make it concrete that there are shared and job-specific parameters (a `map_rect` and a `map_rect_lpdf` would be in Stan). The upside of the more specific route are likely better performance, faster inclusion in Stan and future additions are still possible if one is willing to use zero-sized arguments. Sebastian+Sean to explain / decision on either approach or some process to come to a decision.

### Open Discussion Topics
_Any topics that do not need to be addressed in the short term,
including speculation and brainstorming._

* __Developer name.  Short description.__


Andrew.  We can briefly review all the papers/projects we're working on with Dan, Aki, etc.  I think there are about 10 of them (GMO, hlm's, weak priors, simulation-based checking, Bayesian workflow, a few more I can't remember right now).

Andrew.  Stan demo and movie.
