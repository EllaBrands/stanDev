### Roundtable
_Short (less than 1 minute) description of work in the past week._

- 5/25. Daniel: working on incorporating logging into `stan-dev/stan`. Should have a pull request ready by next week. (I've been really short on sleep lately and most likely will not stay for the meeting to try to take a nap. Didn't know where else to drop this.)


### Unresolved Topics
_Address any open topics from the past meeting._

### New Topics
_New topics that should be addressed immediately or by the next
meeting._

* __Developer name.  Short description.  Desired resolution.__

- Carpenter.  We need a process for pull request reviews so that we don't wind up fighting about formatting, doc, and testing on every single request;  this needs to be across repos, not just one repo at a time, or we'll all go insane (sorry if this is in the wrong place---still don't understand New Topics vs. Open Discussion---this is important, not a long term rambling issue, but maybe not decidable in session in the next meeting---please move if this is in the wrong place).  Agreement and process document.

Betancourt.  Developer interest in Boston-area physics workshop in the fall? Yes/no.

Margossian. What is the best approach to compare the efficiency of two ODE solvers / more generally Stan functions? I've written tests in C++ that use the clock() function. But the measurements I get for CPU times do not agree with what I observe when I run the full Stan model (i.e. using time required to produce 1000 independent samples as a metric).

### Open Discussion Topics
_Any topics that do not need to be addressed in the short term,
including speculation and brainstorming._

* __Developer name.  Short description.__

* Sebastian. Status on current mpi prototype. Mpi into Stan? Should we start to design this?


