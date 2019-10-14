The Stan Algorithm Library provides access to three algorithms — an implementation of dynamic Hamiltonian Monte Carlo, the Automatic Differentiation Variational Inference algorithm, and an implementation of the (Limited Memory) Broyden–Fletcher–Goldfarb–Shannon algorithm.  Each of these algorithms consumes a given target distribution specified as a probability density function, but all three feature different return types with different guarantees.  These different guarantees define the API/client contract, and consequently important policies such as testing burdens.

# Dynamic Hamiltonian Monte Carlo

The workhorse of the Stan Algorithm Library is a state-of-the-art implementation of dynamic Hamiltonian Monte Carlo which belongs to the class of Markov chain Monte Carlo algorithms.  These algorithms return sequentially-correlated parameter values that define estimators of expectation values with respect to the given target distribution.  Under certain circumstances these estimators satisfy a central limit theorem, in which case they will be approximately unbiased with respect to the exact expectation values with quantifiable variance.

Seemingly irrelevant or insignificant details, like the exact implementation of floating point arithmetic, the order of floating point operations, or the consumption of pseudo random number generator states, propagate to significant changes in realized Markov chains.  Consequently bitwise reproducibility of Markov chain output cannot be guaranteed across operating systems, C++ compilers, or computer hardware.

Ensemble and expectation behavior of these Markov chain realizations, and that of their corresponding Markov chain Monte Carlo estimators, also cannot be guaranteed in general.  In the ideal circumstances that enable a Markov chain Monte Carlo central limit theorem, however, we can approximately anticipate the distributional behavior of these estimators relative to the exact expectation values.  In particular, if the effective sample size is estimated accurately then for any real-valued function f,

( \hat{f} - \mathbb{E}[f] ) / \hat{MCMC-SE} ~ normal(0, 1),

where 

\hat{MCMC-SE} = \sqrt{ \hat{\Var[f]} / \hat{ESS[f]} }.  Technically there is a small correction on the order one over the number of iterations that we are ignoring here, but it is negligible for sufficiently long chains.

Although we cannot identify when the necessary ideal circumstances hold, or prove when that they hold in a given problem, we can identify obstructions to these ideal circumstances that indicate that the above results cannot be expected.  These obstructions are codified in diagnostics, in particular 

- The potential scale reduction factor
- Estimated effective sample size per iteration
- Divergent iteration count
- The Energy Fraction of Missing Information 

Optimistically the API assumes that if none of these diagnostics--as implemented in the Stan Algorithm Library--fail for the default algorithm settings, in particular 4 chains, each with 1000 warmup iterations and 1000 main sampling iterations, then the Markov chain Monte Carlo central limit theorem holds.  In particular we can build tests of the dynamic Hamiltonian Monte Carlo output against known, exact expectation values with respect to target distributions that fit without diagnostics failures.

If the ideal circumstances are violated then we cannot guarantee the exact behavior of the Markov chains generate from the given target distribution.  Consequently the API does not guarantee any behavior of any fits accompanied by failures of any of the above diagnostics.

There is one partial exception.  The `adapt delta` configuration, which defaults to 0.8, sets the target criteria for the step size adaptation.  Regardless of the diagnostic failures, increasing this configuration will yield lower step sizes in expectation.  For sufficiently long warmup (at least the 1000 default samples) and reasonable values of `adapt delta` below 0.99, this expectation behavior will typically manifest in the same monotonic behavior for realizations of the adapted step sizes, as well.

# Automatic Differentiation Variational Bayes

Automatic Differentiation Variational Bayes returns a Gaussian approximation to the target probability density function, at least when the target parameter space is unconstrained.  If there are constrained parameters then the algorithm returns a Gaussian approximation to the pushforward target probability density function on an unconstrained parameter space.  This approximate Gaussian probability density function can then be used to construct estimates of target expectation values, typically through Monte Carlo or Importance Sampling estimators.

As with Hamiltonian Monte Carlo, the resulting Gaussian approximations cannot be guaranteed to be exactly reproducible across operating systems, C++ compilers, or computer hardware.  Because the variational optimization utilized by the algorithm internally employs Monte Carlo estimators that consume random state, the differences across these settings can also be quite large. 

Currently there are no conditions under which any target expectation value estimators returned by Automatic Differentiation Variational Bayes will have known behavior that can be used for testing.  This limits testing to unit tests and regression tests.

# Limited Memory Broyden–Fletcher–Goldfarb–Shannon

The (Limited Memory) Broyden–Fletcher–Goldfarb–Shannon algorithm is an optimization algorithm that estimates a single optimum of a specified objective function.  

Per usual, any estimated optima cannot be guaranteed to be exactly reproducible across operating systems, C++ compilers, or computer hardware.

For well-behaved objective functions, including but not limited to unimodality and twice differentiability, any optimum estimated by the (Limited Memory) Broyden–Fletcher–Goldfarb–Shannon algorithm is guaranteed to converge to the true optimum within certain precision bounds.  For example explicit bounds can be constructed for purely concave objective functions.  These special cases can then serve as well-defined, if limited, tests of the algorithm implementation.