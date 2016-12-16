One of the strongest motivations for Stan is to help users build the models they want to build instead of having to choose from a limited set of default models that do not capture the particular structure of their analysis.  That freedom, however, has consequences.  The more complex a model the more possibilities there are to implement it incorrectly, and the harder it will be for Stan to fit accurately.  Consequently users have to take care to make sure that they use Stan _responsibly_!

Robust statistical practice is a lifelong endeavor that continuously evolves as we learn more, but below are a list of best practices that will help you accelerate you along that evolution as quickly as possible.

# 1. Treat your Stan program as software

1a.  Maintain reproducibility by saving the model, data, 
       and inits in files and the RStan commands in scripts.

1b.  Use version control on your files and scripts.

# 2. Start simple

2.    Start simple!  Build your model in stages, ensuring
      good fits at each stage.

# 3. Think Generatively

# 4. Validate your fit

## Try to recover simulated values

3.    Fit your model to simulated data to ensure that
       Stan can recover the true values.

## Check generic diagnostics

## Check algorithm-specific diagnostics

4.    Keep an eye on those diagnostics!