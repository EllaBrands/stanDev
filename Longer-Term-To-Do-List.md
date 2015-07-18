This list is meant to complement the issue tracker and help prioritize projects.

#### Stan 3.0

We need to gather up all the backward-compatiblity breaking interface changes and do them at once as a Stan 3 rollout. 

#### C++11 Migration

We really want to move to using C++11 for both code cleanliness and efficiency, but we're held up by CRAN and I believe by Python 2.x.  

#### Command Refactor

This is required to take pressure off the interfaces and provide more uniformity.  We're being held up in basic design w.r.t. how much C++ we want to expose in the form of callbacks and how flat vs. hierarchical to make the commands.

#### Protocol Buffer Transport Layer

This is related to command refactoring and will probably live on top of it.  

#### Stan Book

Need to figure out 
- target audience and level of stats background we assume
- whether to have it depend on a given interface for concreteness or to make it interface neutral with appendices

#### CloudStan

Something to run models in the cloud.

* user-defined models
* pre-existing models with user data

#### Higher-Order Autodiff

Finishing Stan's existing <code>fvar</code> implementation.  Many projects depend on this, including MLE, RHMC, MML, VB, EP, 

#### Maximum Marginal Likelihood

Including both the fitting and the 

#### Stiff ODE Solver

Integrate either 

- fasta: simpler, it looks like, and what deSolve does in R
- Sundials: more feature rich, what Sebastian's urging us to use

Then 
 
- bigger coupled system with our own autodiff

#### Implicit Functions and Differential Algebraic Equations (DAE)

Ben's been asking for the first for years.  The deSolve package in R does both, I think.

#### Black-Box Variational Inference (BBVI)

Get the basic BBVI from Alp tested and integrated into Stan's commands

#### Stochastic Black-Box Variational Inference (SBBVI)

Extend BBVI to stochastic updates
    - requires updates

#### Black-Box Expectation Propagation (BBEP)

#### Data-Parallel Black-Box Expectation Propagation (DPBBEP)

#### Ragged Arrays and Matrices

This includes ragged arrays and arrays of matrices of different sizes.

#### Sparse Matrices

This requires integrating the Eigen sparse matrix lib.

#### Structured Matrices

Need these for scalable Gaussian processes.  The goal for Andrew is to code Aki's birthday model in Stan.

#### Riemannian-Manifold Hamiltonian Monte Carlo (RHMC)

This is waiting on higher-order autodiff.

#### Higher-order Autodiff Testing Framework

Right now we're bottlenecked on testing for new functions, particular for higher orders.

#### Revise BUGS and ARM Models to Best Practices

It's embarassing that we have lousy models as examples.  Need to go through entire repo and fit.

#### Speed Evaluations vs. BUGS/JAGS

Andrew's been asking for these for years.  It's complicated.

#### Ensemble Samplers

Peter already built Goodman and Weare and has evaluations, and I think the evals are all there.  

- write up paper

Even better would be

- integrate into Stan

The issue with the integration is how to deal with the ensemble samples given that they don't form Markov chains and otherwise have to be used in aggregate, which is OK for expectations of parameters in model, but not for anything else.

#### Metropolis

We can add this, but we never figured out any decent adaptation.

#### Pre-Specified Mass Matrices

Right now, we only create them via adaptation.  Requires working out a data format for them, though we should probably use whatever we use for inits and data.

#### Log CDFs, CCDFs, truncation

We need to fix all of the CDFs and CCDFs so they have specialized code for the log scale.  It's probably not a high priority, because they will underflow or overflow derivatives in most cases.  We also need specialized code for `log(foo_cdf(H, theta) - foo_cdf(L, theta))` that removes redundant calculations and is more robust for `[L,H]` truncation. 

#### Inverse CDFs

All of them that we can implement.  We need them for copulas and other constructions, including reparameterizations.

#### Complex Number Support

Make sure that constructors for `std::vector<stan::math::var>` do the right thing and programs using them in Eigen, etc., do the right thing.


