This list is meant to complement the issue tracker and help prioritize projects.

#### Stan 3.0

We need to gather up all the backward-compatiblity breaking interface changes and do them at once as a Stan 3 rollout. 

#### C++11 Migration

We really want to move to using C++11 for both code cleanliness and efficiency, but we're held up by CRAN.

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


#### Maximum Marginal Likelihood

Including both the fitting and the evaluation of how well the variance approxmations work.  Andrew wants to use his "GMO" algorithm.


#### Implicit Functions and Differential Algebraic Equations (DAE)

Ben's been asking for the first for years.  The deSolve package in R does both, I think.

#### Stochastic Black-Box Variational Inference (SBBVI)

Extend BBVI to stochastic updates
    - this should be doable by modifying the data in an appropriate Stan program

#### Black-Box Expectation Propagation (BBEP)

#### Data-Parallel Black-Box Expectation Propagation (DPBBEP)

#### Ragged Arrays and Matrices

This includes ragged arrays and arrays of matrices of different sizes.

#### Sparse Matrices

This requires integrating the Eigen sparse matrix lib.

#### Structured Matrices

Need these for scalable Gaussian processes.  The goal for Andrew is to code Aki's birthday model in Stan.

#### Null space type. See stan-dev/stan#380

Implement a vector type that satisfies

A * u = 0

for a given (k by n) matrix A.

Algorithmically, we need to compute the SVD of A then store the (n - k) rightmost vectors of V, v_{i}. The unconstrained parameters are then the coefficients of the expansion

u = sum_{i} \alpha_{i} v_{i}.

Because A is constant, the Jacobian is constant and the gradients are simple.

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

#### User-Defined Transforms with Jacobians

Allow users to write a transform 

```
vector my_transform(vector theta, vector x, int[] x_int);
```

for which we use higher-order autodiff to compute Jacobian determinant and add to the log density.  It'd compile to a function that takes an accumulator or variable to update or something that returns a struct with the value and the Jacobian.  Ideally, we should allow all these data arguments to be empty --- they're easily inferrable by default (here and in ODEs).

#### Way to Turn off Warnings

Sometimes a user knows they don't need a Jacobian or have supplied one.  Sometimes they want integer arithmetic.  We should have a way within a Stan program to turn off warnings.  The issue is that false positives mask true positives for warnings.

#### Parallel Model Evaluation

In many models, such as PK/PD random-effects models, the density comes in natural blocks that could be evaluated in parallel.  The trick is going to be figuring out how to factor the density definition so that components of it can be tackled in parallel.

#### Caching difference in log probability (with constants vs without)

See discussion in issue [#584](https://github.com/stan-dev/stan/issues/584), but here's the gist: should we calculate the log prob with and without constants once, cache the difference, and then use this to initialize `lp__` in subsequent iterations?

#### Syntax Checker --- Lint, etc.

Add a lint-like or warnings in a syntax checker.  Issue to cover include:
* variables being used before they're defined
* inv_gamma, gamma with small arguments
* normal with large scale

#### Geometry research

Figure out if different transforms have different behaviors for sampling.  From @betanalpha:  My intuition is that the well-posed transformations will all end up being equivalent (as we’ve seen with the simplex transformations) but it’s definitely worth exploring and writing up. The whole “map to R^{n}” is  an important principle that is rarely discussed in the statistical literature  but is crucial for efficient algorithms. 

#### Run until convergence

Figure out how to do online monitoring of convergence so we can run until we converge without specifying number of iterations.  Need to set thresholds so we don't get bias due to early stopping (not even sure this is possible, but if not, we want to understand why).

#### Model testing framework

This is high priority, but we don't know how to do it.  We want to be able to test that Stan programs
* get the "right" answer to within MCMC error
* make sure we have speed regression tests so we don't get slowdowns by surprise

#### CSV I/O

Andrew wants this, but no idea how to do it given that CSV format isn't very general.  We'll probably need to read multiple file types at once if we want this to work.

#### Copulas via Higher Order Autodiff

(Copied from closed issues https://github.com/stan-dev/math/issues/42 and https://github.com/stan-dev/stan/issues/938).  Next step is to start a Wiki for design and then when we have something in mind create a concrete issue.  Here's what @bgoodri said:

In the case of a copula, we know the explicit form of the multivariate CDF of the marginal uniforms

```
F(u_1, u_2, ..., u_d | theta)
```
but for Stan's purposes we need the (logarithm of the) copula density function, which is

```
dF(u_1, u_2, ..., u_d | theta)
------------------------------ = f(u_1, u_2, ..., u_d | theta)
du_1,du_2,...,du_d
```

but for large d, there is sometimes no explicit form of `f(u_1, u_2, ..., u_d | theta)`. So, loosely speaking, it would be great if we could evaluate 

```
var f_log(var_vector u, var theta) {
  const int d = u.rows();
  var p = F(u, theta);
  var dp_du_1 = get_partial(p, u[1]);
  var dp_du_1_du_2 = get_partial(dp_du_1, u[2]);
  /* more of these */
  var log_density = log(get_partial(dp_du_*, u[d]);
  return log_density;
}
```

And what @syclik replied:

We can do this with second-order autodiff.  But it's not going to be super efficient in high dimensions, because each dimension is going to involve a forward pass on the function F, unless we can figure out how to do nested evaluation the way we are building up the ode solver.

The `get_partial()` operation you have will be a forward-mode auto-diff d_ value.  That can then be logged, or whatever, and autodiffed through for HMC.

If the F doesn't involve any pdf or cdf calls in Stan, could do it now.  Otherwise, we're going to have to wait until the 2nd order autodiff is completed.  It's held up on Windows testing at the moment.

Are there a fixed number of these, or do people write their own F?  

#### Testing prob functions, cdfs, ccdfs and log forms

Originally from https://github.com/stan-dev/math/issues/357, where @bob-carpenter said:

Add all the missing tests.  We need to make sure that changing the answer breaks a test.  It's impossible to refactor these functions without tests and they need it badly to remove redundant calculations.

First step is to enumerate which tests are needed by poring over each of the functions.  Daniel mentioined a lot of these are already handled in the testing framework, but we need some doc on how to use that and need to review it to make sure it covers everything.

#### Discrete Parameters

So far, we don't know how to tackle this one.  We'd ideally like to be able to compute the Markov blanket of a graphical model and do Gibbs, but Stan's language is too general for that.  One approach would be a sublanguage for graphical models.  Another approach would be to just tack on Gibbs or Metropolis as is, but we don't know how to pull out the conditionals for Gibbs or the marginal of the discrete parameters for Metropolis without evaluating the whole density.




