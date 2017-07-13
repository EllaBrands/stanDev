### Roundtable
_Short (less than 1 minute) description of work in the past week._


### Unresolved Topics
_Address any open topics from the past meeting._

### New Topics
_New topics that should be addressed immediately or by the next
meeting._

* __Developer name.  Short description.  Desired resolution.__

### Open Discussion Topics
_Any topics that do not need to be addressed in the short term,
including speculation and brainstorming._

* __Developer name.  Short description.__

* Andrew Gelman and Jon Zelner.  Inference for models with discrete states as occur in epidemiology. (edit from Betancourt: these models often require inference over tree spaces).

* Sebastian. MPI design 1: How to make (transformed) data permanent and referable by some symbol which can be serialized over the network. That is, the root node must be able to tell the workers "take data X" and that "X" is referring to data which is already on the worker. Can we use flyweight singletons?

* Sebastian. MP design 2: Given that data should be local, we have to stick this stuff into stan (or into stan-math if we include the solution of design question 1 into stan-math). If it goes into stan...how and where/is that OK given we integrate things deeply with the AD stack?

* Andrew.  Speed comparisons for Stan optimize for simple linear and logistic regressions.

* Ben. Function names in Stan3

* Breck. Governance