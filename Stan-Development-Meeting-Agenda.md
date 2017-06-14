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

Breck. Proposal to adopt approach from Producing Open Source Software, Karl Fogel (http://producingoss.com/) as the framework for Stan development. 

Andrew.  Diagnostics and adaptation for approximate algorithms.

Andrew. Should we consider putting this into Stan:  Jeffrey Regier, Michael I. Jordan, Jon McAuliffe
Fast Black-box Variational Inference through Stochastic Trust-Region Optimization
https://arxiv.org/abs/1706.02375

Aki writes:  "They have implemented it in Stan and they compare it to ADVI using 182 different models. "For 111 of the 182 models (61%), on sets of 5 runs, the mean optimum found by ADVI and TrustVI did not differ significantly. TrustVI converged to significantly better optimal values on 60 models (33%). ADVI found better optimal values for 11 models (6%)."

My thought would not be to put it in Stan right away.  But should we _consider_ putting it into Stan, running tests on the existing rstan models, etc?