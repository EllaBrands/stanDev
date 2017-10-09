One of the strongest motivations for Stan is to help users build the models they want to build instead of having to choose from a limited set of default models that do not capture the particular structure of their analysis.  That freedom, however, has consequences.  The more complex a model the more possibilities there are to implement it incorrectly, and the harder it will be for Stan to fit accurately.  Consequently users have to take care to make sure that they use Stan _responsibly_!

Robust statistical practice is a lifelong endeavor that continuously evolves as we learn more, but below are a list of best practices that will help accelerate you along that evolution as quickly as possible.

# 1. Treat your Stan program as software

Your model is only as good as your ability to communicate it to others, and communication is drastically improved by treating your Stan program as a piece of software and applying the best practices from the software development community.  This helps not only to promote reproducibility but also to facilitate prompt answers from the Stan community when you run into problems.

Write your program to be easily read.  Use indentation to separate blocks of code, pull repeated calculations into reusable functions, and give variables and functions meaningful names.  With well-formatted code and well-chosen names simple Stan programs will be understandable without any comments, but more sophisticated models may require comments describing complex manipulations or calculations.

Facilitate reproducibility by saving the Stan program, data, and inits in separate files so that anyone can fit your model in any of the Stan interfaces.  Saving data and inits is simplified with the ```stan_rdump``` and ```read_rdump``` functions provided by some of the interfaces -- see the individual interfaces for more information.

Ideally, your analysis will be broken up into a Stan program, a data file, possibly an init file, and scripts that process the analysis in your favorite environment.  Because each of these files will constantly evolve as you improve your analysis, we highly recommend using version control software such as ```git``` and supporting services like GitHub.  Hosting your analysis on GitHub is free and not only provides a backup of your project but also a history of all of your changes in case you need to revert any bad updates.

# 2. Think generatively

The ideal statistical model is a _generative model_ that describes the data generating process forward from the underlying parameters all the way to the measurement and, ultimately, the raw data.  Reason about how the data were generated with the experts who collected the data, breaking the process up into sequential steps that can be modeled piece by piece.

Data "corrections" such as background subtraction and deconvolution censor information in the data and limit how much you can learn in a given experiment.  On the other hand, modeling the data generatively as a sum of signal plus background or signal smeared with measurement variability allows you to build an accurate model capable to extracting as much information from the data as possible.

# 3. Start simple

As important as generative modeling is, it's also important to start simple.  Instead of trying to model every detail of the experiment immediately, begin with a simple approximation that captures the gross features of the data generating process.    Relative to long, complex models, simple models are much easier to communicate and drastically easier to debug.  

Once you can fit the initial simple model you can add more structure in stages, making sure that you can fit the model after each addition.  In this way you can identify problematic structures as you add them, instead of trying to find them amongst the many possible structures in the full model.

# 4. Validate your fit

You want to know the dirty truth?  Statistical algorithms don't always give you the right answer.  Even Stan.  If you can't verify that your statistical algorithm is giving an accurate answer then you can't tell whether a bad fit is due to the model not capturing the data generating process or your statistical algorithm returning garbage!

Here we will focus on verifying the output of Markov chain Monte Carlo, especially with the Hamiltonian Monte Carlo algorithm used in Stan.  All of these criteria are necessary but not sufficient conditions for a good fit -- in other words they all identify problems that will ensure a bad fit but none of them can guarantee a good fit.

## Recover simulated values

One of the most powerful means of validating a statistical algorithm is to verify that you can recover the ground truth from simulated data.  Begin by selecting reasonable "true" values for each of your parameters, simulating data according to your model, and then trying to fit your model with the simulated data.  A nice consistency check is whether or not the 5% to 95% marginal posterior interval captures each of the "true" parameter values.  If your fit does not pass this check then something is going wrong with the statistical algorithm and your model, and you will not be able to trust it when applied to real data.  

The simulated data itself can be generated in Stan using the random number generators in the generated quantities block or using random number generators in your favorite computing environment like R or Python.  The simulation process itself will follow directly from the model, although if you have not focused on building your model generatively then it may not be easy to see that!  

## Check generic diagnostics

The best generic diagnostic for the accuracy of a Markov chain Monte Carlo algorithm is the split R-hat statistic which quantifies the consistency of an ensemble of Markov chains.  The idea is to run multiple Markov chains, 4 is an okay default but running more makes the diagnostic more sensitive, and ensure that they're all exploring the same regions of parameter space.  **If even one chain is inconsistent with the others then all of the chains are suspect!**  In practice we have found that requiring ```Rhat < 1.1``` is a good default requirement for each parameter.

Another potential problem to keep in mind is Markov chains that explore very slowly.  Slow chains lead to low numbers of effective samples, and if there are too few effective samples then we can't accurate estimate the number of effective samples at all.  A good check for such issues is the number of effective samples per iteration -- if ```N_eff / N < 0.001``` then you should be suspect of the effective sample size calculation.

## Check algorithm-specific diagnostics

Hamiltonian Monte Carlo provides not only state-of-the-art sampling speed, it also provides state-of-the-art diagnostics.  Unlike other algorithms, when Hamiltonian Monte Carlo fails it fails sufficiently spectacularly that we can easily identify the problems.

In practice this means checking that there are no _divergence_ in your fit.  Divergences are automatically reported in some interfaces, namely RStan, but they can be directly access in the fit output in all interfaces.

There is an additional _energy_ diagnostic in development that will be made available to the interfaces soon.

# 5. Heed the Folk Theorem

The Folk Theorem of Statistical Computing is a heuristic developed by Andrew Gelman stating that most statistical computational problems are due not to the algorithm being used but rather the model itself.  For example, a poorly-identified model with diffuse priors will require sojourns into the extremes of parameter space that will break even the best statistical algorithms.  Instead of trying to build an algorithm that can handle such extreme values, we can simply replace the diffuse priors with weakly-informative priors and use the existing algorithm.

The Folk Theorem is an important perspective because if forces the user to confront and evaluate the assumptions inherent to their model.  This is especially useful when adhering to a sequential model building process where problematic model structures are easy to identify.  Moreover, the Folk Theorem provides continued motivation for generative modeling -- in practice you will find that many modeling problems are non-generative in nature, and replacing that structure with a generative component almost always provides significant improvement.