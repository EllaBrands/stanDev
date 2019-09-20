The core of the Stan library is composed of three main components: the Stan language, the Stan math library, and the Stan algorithm library.  The Stan language is a probabilistic programming language for specifying (unnormalized) joint probability density functions over unbound parameter variables and bound data variables.  The Stan math library complements these models with automatic differentiation, algorithmically evaluating gradients of the specified model with respect to the unknown parameter variables.  The Stan algorithm library then provides Bayesian computational methods that consume the joint density function and its gradient to estimate inferences.

A priority of the Stan algorithm library is the computation of _faithful_ inferences from sophisticated models.  Formally inferences are defined as posterior expectation values; questions about the model are framed as functions over the parameter space and their answers are given by the corresponding posterior expectation.  Faithful methods return not only estimators of these expectation values but also accurate quantification of estimator error so that the utility of those estimates can be judged in a principled way.   Without this error quantification there is no way for a user to determine how much a method might be corrupting the inferences from the desired model.  Robust methods maintain reasonable error over a wide class of models, especially as the models incorporate higher-dimensional parameter spaces and more complex structure.

Faithful computation allow users to focus on modeling building instead of computation, but it is an incredibly challenging goal and ultimately restricts the scope of Stan.  It not only disqualifies many methods popular in statistics and machine learning but also limits the scope of the models that can be considered.  For example members of Stan development team are not aware of any methods robust enough to provide faithful computation for non-trivial models defined over structured discrete spaces such as networks and phylogenies, or models that combine continuous and discrete spaces.

Because of this priority we consider algorithms for inclusion into Stan only once they are mature enough to be supported by a reasonable amount of theory and have undergone extensive testing.  With so many algorithms introduced each week the Stan development team unfortunately does not have the resources to assist with this testing, and consequently the evidence burden is the responsibility of anyone who might want their method to be considered.

# Baseline Algorithms

The baseline algorithm in Stan is a state-of-the-art implementation of dynamic Hamiltonian Monte Carlo.  Hamiltonian Monte Carlo has key theoretical advantages that ensures not only efficient computation but also accurate computation.  This method exploits the gradient of the target posterior density function to simulate measure-preserving flows that rapidly explore the typical set of the target distribution even high dimensions.

Under ideal conditions dynamic Hamiltonian Monte Carlo estimators satisfy central limit theorems that allow us to accurately estimate their error online.  Recent theoretical work has demonstrated that these necessary conditions are weaker than those needed for other Markov chain Monte Carlo algorithms, explaining the empirical robust of the method even for complex, high-dimensional models.  Most importantly, however, when these conditions are violated the sampler behaves idiosyncratically which allows us to clearly diagnose problematic estimation.  In other words when users respect the Stan diagnostics they can determine with high confidence when they can trust the faithfulness of their Markov chain Monte Carlo estimation.

The other algorithm in Stan is Automatic Differentiation Variational Inference, an implementation of variational Bayes that also exploits the gradients of the target posterior density function.  Unlike Hamiltonian Monte Carlo, however, this method has little theory to support its performance in practical settings.  In particular there are currently no methods for empirically quantifying the errors of the these variational Bayesian estimators aside from comparing to methods like dynamic Hamiltonian Monte Carlo.  Recent work reframing variational Bayes as a precursor to importance sampling methods has provided preliminary error estimation and diagnostics, https://arxiv.org/abs/1802.02538, but so far this work has largely demonstrated that variational Bayes is too fragile for most practical problems.

To calibrate expectations, under our current criteria this implementation of variational Bayes would not be mature enough to be considered for inclusion.

# Criteria for Proposal Consideration

Members of the Stan development team are active participants of the statistics and machine learning communities and are constantly appraising and experimenting with new methods that might have the potential to complement or compete with dynamic Hamiltonian Monte Carlo.  That said we will inevitably miss something, and so we welcome the proposal of new methods with sufficient theoretical and empirical validation that we might have overlooked.

From our perspective a mature algorithm warranting consideration must have conceptual, theoretical, and empirical support.  If you believe that you have an algorithm with supporting evidence in all three categories then you begin a formal inclusion request by forking https://github.com/stan-dev/design-docs, adding a design document detailing your algorithm and its supporting evidence, and submitting a pull request.  We also recommend that you announce the design document on the Stan Forums, https://discourse.mc-stan.org, under the Algorithms topic.

## Conceptual

Why does the proposed method work?  What information about the target probability density function does it exploit, and why does that information ensure robust estimation for sophisticated, high-dimensional models?  

What assumptions about the target probability density function does the method assume, and what happens qualitatively to the practical behavior of the algorithm when those assumptions are violated?

## Theoretical

Under what formal conditions will the proposed method return accurate estimation of expectation values?  How does the method perform with only finite computational resources or small data sizes?

Can the estimator error be theoretical bounded or otherwise constrained?  Can that bound be refined using empirical information available when running the method?

How does the structure of the method interact with the structure of a given target distribution, and how does that interaction affect the estimator error or bounds on the estimator error?  For what classes of target probability density functions will the method work well, and what classes of target probability density functions will frustrate the method?

How does the method behave when working well and when struggling?  What behaviors, if any, can be used to diagnose underperforming applications of the method?

What degrees of freedom does the method have?  How do they interact with a given target probability density function and how can they be tuned in a given application?  How sensitive is the performance of the method to these degrees of freedom?

## Empirical

How well can the method estimate known expectation values?

For empirical testing one has to consider a wide class of models that are reasonably representative of those at the frontiers of Stan applications, especially those with complex structure and higher dimensions.  

When exact posterior expectation values are unknown, Simulation-Based Calibration, https://arxiv.org/abs/1804.06788, can be used to qualify the error within a given class of models.  Although this does not yield precise error bounds it is very sensitive to many of the pathologies that are known to manifest in fragile algorithms and hence provided powerful qualitative evidence.

If exact posterior expectation values are known, or they can be computed to high precision with verified computation, then the error can be computed directly.  A preliminary benchmark of this form can be found at https://github.com/stan-dev/stat_comp_benchmarks.  This benchmark assembles a suite of low-dimensional but nontrivial models with known expectation values, mostly for parameter functions, that can be used to evaluate an external algorithm.  Errors are computed for each expectation value and then compared to the corresponding variances to flag any that are unsatisfactory large.

This benchmark provides an extremely weak test of a computational method and is no where near sufficient for consideration as a contribution to Stan.  In particular it does not quantify how well an algorithm can recover variances and it does not calibrate estimator errors.  Moreover the models provided are simple relative to the sophisticated models at the heart of most Stan analyses.  

More realistic benchmarks have not yet been needed as nothing has passed this test sufficiently well for further consideration.  Much more elaborate benchmarks are under development by groups affiliated with the Stan development team and when they are finalized they will be included here.

# Recommended Algorithm Development Practices

Because of limited resources the Stan development team is unfortunately not able to support the development or evaluation of external methods.  We will do our best to comment on potential approaches based on our availability, especially on the Stan Forums, but we cannot guarantee that we will be able to answer every question.  

To facilitate testing we recommend that you fork the main Stan repository, https://github.com/stan-dev/stan, and implement your method along with associated service routes.  This allows you to run preexisting tests and benchmarks through interfaces like CmdStan.

Finally, for any functionality that builds upon any of the Stan libraries an independent package is always an alternative to this process.  The flexible BSD license of the core Stan libraries gives third-party developers complete control for how to incorporate or extend Stan, and such packages are useful ways to solicit feedback and evaluation from larger audiences.  Successful third-party packages using Stan include arviz (statistical visualization and analysis in Python), Prophet (time series forecasting in Python and R ), and StataStan (embedded probabilistic programming language in Stata).