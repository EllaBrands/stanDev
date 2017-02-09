### Roundtable
_Short (less than 1 minute) description of work in the past week._

### Unresolved Topics
_Address any open topics from the past meeting._

### New Topics
_New topics that should be addressed immediately or by the next
meeting._

* __Developer name.  Short description.  Desired resolution.__

Betancourt.  Merging algorithm changes.  Establish an actual policy instead of vague agreements that end up falling apart on the first merge.

Carpenter.  What do we want to do with vector views, matrices, etc.  Decision about what to do (isn't that always the desired resolution)? [ed: Be specific on "what to do" -- as is this sounds extremely vague. Checklists, etc welcome].

Sebastian. Parallel ODEs in Stan using OpenMP (linear speedup with # CPUs!): It works by hijacking the ODE facilities. Do we want this in Stan to be available to all users? If yes, then: Need for ODE system refactor (stiff+non-stiff)? Who wants to join? More work needed: Analytic Jacobians of ODE rhs (optionally with code-generation); ragged arrays.

### Open Discussion Topics
_Any topics that do not need to be addressed in the short term,
including speculation and brainstorming._

* __Developer name.  Short description.__

Andrew.  Nonidentifiability in symmetric probabilistic matrix factorization model.

Andrew.  Priors as part of parameter specification in Stan language?