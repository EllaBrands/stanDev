One of the strongest motivations for Stan is to help users build the models they want to build instead of having to choose from a limited set of default models that do not capture the particular structure of their analysis.  That freedom, however, has consequences.  The more complex a model the more possibilities there are to implement it incorrectly, and the harder it will be for Stan to fit accurately.  Consequently users have to take care to make sure that they use Stan _responsibly_!

Robust statistical practice is a lifelong endeavor that continuously evolves as we learn more, but below are a list of best practices that will help you accelerate you along that evolution as quickly as possible.

# 1. Treat your Stan program as software

Your model is only as good as your ability to communicate it to others, and communication is drastically improved by treating your Stan program as a piece of software and applying the best practices from the software development community.  This helps not only to promote reproducibility but also to facilitate prompt answers from the Stan community when you run into problems.

Write your program to be easily read.  Use indentation to separate blocks of code, pull repeated calculations into reusable functions, and give variables and functions meaningful names.  With well-formatted code and well-chosen names simple Stan programs will be understandable without any comments, but more sophisticated models may require comments describing complex manipulations or calculations.

Facilitate reproducibility by saving the Stan program, data, and inits in separate files so that anyone can fit your model in any of the Stan interfaces.  Saving data and inits is simplified with the ```stan_rdump``` and ```read_rdump``` functions provided by some of the interfaces -- see the individual interfaces for more information.

Ideally, your analysis will be broken up into a Stan program, a data file, possibly an init file, and scripts that process the analysis in your favorite environment.  Because each of these files with constantly evolve as you improve your analysis, we highly recommend using version control software such as ```git``` and supporting services like GitHub.  Hosting your analysis on GitHub is free and not only provides a backup of your project but also a history of all of your changes in case you need to revert any bad updates.

# 2. Think generatively

The ideal statistical model is a _generative model_ that describes the data generating process forward from the underlying parameters all the way to the measurement and, ultimately, the raw data.  Reason about how the data were generated with the experts who collected the data, breaking the process up into sequential steps that can be modeled piece by piece.

Data "corrections" such as background subtraction and deconvolution censor information in the data and limit how much you can learn in a given experiment.  On the other hand, modeling the data generatively as a sum of signal plus background or signal smeared with measurement variability allows you to build an accurate model capable to extracting as much information from the data as possible.

# 3. Start simple

As important as generative modeling is, it's also important to start simple.  Instead of trying to model every detail of the experiment immediately, begin with a simple approximation that captures the gross features of the data generating process.  

Relative to long, complex models, simple models are much easier to communicate and drastically easier to debug.  Once you can verify that the simple model is being fit correctly then you can evaluate the validity 

2.    Start simple!  Build your model in stages, ensuring
      good fits at each stage.



# 4. Validate your fit

## Try to recover simulated values

3.    Fit your model to simulated data to ensure that
       Stan can recover the true values.

## Check generic diagnostics

## Check algorithm-specific diagnostics

4.    Keep an eye on those diagnostics!