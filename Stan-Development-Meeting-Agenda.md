### Roundtable
_Short (less than 1 minute) description of work in the past week._

### Unresolved Topics
_Address any open topics from the past meeting._

### New Topics
_New topics that should be addressed immediately or by the next
meeting._

* __Developer name.  Short description.  Desired resolution.__

Betancourt.  Merging algorithm changes.  Establish an actual policy instead of vague agreements that end up falling apart on the first merge.

Sebastian. Parallel ODEs in Stan using OpenMP (linear speedup with # CPUs!): It works by hijacking the ODE facilities. Do we want this in Stan to be available to all users? If yes, then: Need for ODE system refactor (stiff+non-stiff)? Who wants to join? More work needed: Analytic Jacobians of ODE rhs (optionally with code-generation); ragged arrays.

Aki. Mike made a proposal on adding GP covariance functions with cov, chol, lpdf. In discourse there was some dicussion. I have couple questions.

Here's the proposal document: https://github.com/stan-dev/stan/wiki/Adding-a-Gaussian-Process-Covariance-Function (Rob added)

### Open Discussion Topics
_Any topics that do not need to be addressed in the short term,
including speculation and brainstorming._

* __Developer name.  Short description.__

Andrew.  Nonidentifiability in symmetric probabilistic matrix factorization model.

Andrew.  Priors as part of parameter specification in Stan language?