# Stan Frequently Answered Questions (FAQ)

This is more of a collection of answers than questions hence the name.  It might be better to reformulate it eventually so users can index by question rather than by answer.

- *Variables must be defined before they are used.*  Otherwise, their value will be not-a-number (NaN) and using them as arguments to sampling statements or functions will fail.  This will manifest as an inability to initialize or many error messages after initialization.  

- *Sampling statements (`~`) only increment the log density.*  All operands, including the value to the left of the `~`, must be defined before the sampling statement is executed.

- *Constrain parameters to match support.*  Every value for the parameters that satisfies the declared constraints should have finite density (finite log density).  If not, random initialization may fail and sampling or optimization may devolve to rejection sampling and exhibit bias (due to inability to adapt to placement of rejected volumes).  This also means not rejecting or returning negative infinity as a log density as a means of trying to define constraints.  Also, constraints on transformed parameters are meant to be satisfied, not used to reject otherwise legal values of parameters.

- *Do not use interval-bounded priors unless physically necessary.*   Although `<lower = 0, upper = 1>` must be used for parameters representing probabilities, interval bounds tend to provide poor priors both statistically and computationally.  If values beyond the bounds are consistent with the other parameters and data, probability mass will bunch up at the boundaries, biasing estimates and pushing the sampler or optimizer out to plus or minus infinity on the unconstrained scale.

- *Do not use overly diffuse priors.*  In particular, priors like `gamma(1e-4, 1e-4)` or `inv_gamma(1e-4, 1e-4)` lead to the opposite problem of interval bounded priors, namely they tend to pull estimates away from zero toward the tail.  In general, make sure typical values from priors are reasonable for parameters.

- *Parameterization shape matters*.  There are multiple ways to parameterize the same density and some of them are more computationally efficient and arithmetically stable than others, and some produce such challenging posterior geometries that they induce biased samples.  The ideal posterior will be unit scaled (centered at zero, scale of one) and have no correlation among the parameters.  The usual sign that a parameterization needs to be fixed is divergent transitions.  The usual fix is to try to both standardize the scale and break dependencies among parameters in the prior.

- *Scale matters in the parameterization*.  Random initialization is centered around zero on the unconstrained scale.  Warmup begins with the assumption that all values have unit scale.  That is, the parameters are all assumed to be standard normal.  Departurtes from this scale are learned during adaptation, but this can be slow if many variables depart from standard scale.  

- *Posteriors must be proper, so identifiability matters.*  Strict identifiability tends to work better for sampling than soft identification through priors, though soft identification often allows more natural prior parameterizations.

- *Sampling and optimization happen on the unconstrained scale.*  All parameters are transformed to the unconstrained scale for sampling with an appropriate Jacobian adjustment (which is turned off for optimization).  For example, positive constrained parameters are log transformed, interval constrained paramters are log-odds (logit) transformed, simplex parameters are transformed with a steak-breaking construction, and covariance matrices are transformed with a Cholesky factorization.

- *Nonlinear parameter transforms require Jacobians to define distribution.*  If a parameter is transformed with a non-linear function, then given a distribution, it requires a Jacobian adjustment (adding the log absolute determinant of the Jacobian of the inverse transform evaluated at the unconstrained parameters).
- *Basic types for containers matter for linear algebra operations.*  Only vectors, row vectors, and matrices may be used as operands for matrix arithmetic or arguments to linear algebra functions;  returns are all the natural types (e.g., a row vector multiplied by a column vector is a scalar; a matrix times a vector is a vector).

- *There shouldn't be any divergent transitions.*  Divergences happen in Hamiltonian Monte Carlo when numerical stability is detectedin the algorithm used to simulate the Hamiltonian dynamics of a fictional particle representing the parameters, specifically when the simulated Hamiltonian diverges to a value far away from the origianl Hamiltonian (it should be exactly conserved in a perfect simulation).  If divergent transitions arise, first try increasing the target acceptance rate to 0.9, 0.95, or even 0.99 or higher (`adapt_delta` in most interfaces) and also reduce the initial stepsize to 0.1 or 0.01 (`stepsize` in most interfaces).  This will cause slower but surer adaptation and sampling.  If that does not solve the problem, reparameterization will be required to deal with the extreme curvature being found.  A parameterization with less variation across in posterior curvature (value of Hessian matrix of second derivatives of the log density with repspect to the parameters). Plots in the R interface are available to highlight where divergences occur to provide clues for reparameterizations.

- *Efficiency depends on number of functions evaluated*.  There is a cost for each operation to calculate the value, the Jacobian, and to propagate all derivatives from the results to the inputs.  Reuse calculations and vectorize wherever possible.

- *Efficiency depends on the model being reasonably well specified for the data.*  If the model does not fit the data well, efficiency will suffer.  (This is what Andrew Gelman calls "The Folk Theorem".)

- *Push calculations to the transformed data or generated quantity blocks if possible.*  These are evaluated once and once per iteration and do not require derivatives.

- *Memory depends on number of functions evaluated and number of operands*.  There is a total cost in memory of around 24 bytes plus 8 bytes per argument for each function evaluated.  

- *Use specialized functions wherever possible*.  Do not create a diagonal matrix, then multiply---use the specialized multiplication (`diag_pre_multiply`, `diag_post_multiply`, and `quad_form_diag`.

- *Use Cholesky factor parameterizations of correlation matrices and scale.*  Rather than using a covariance matrix parameter or even a correlation matrix parameter, use a Cholesky factor for a correlation matrix and scale with a vector of scales using `diag_pre_multiply`.  Use an LKJ prior on the Cholesky factor of the correlation matrix. (`lkj_cholesky()`).  The correlation and/or covariance matrix may be recovered in the generated quantities block.

- *Code formatting matters for readability and maintainability.*  Use standard formatting, ideally the one we use in the manual.  No tabs, indent two spaces, spaces around braces (`{`, `}`) and arithmetic and comparison operators (`+`, `&&`, etc.),  no space around parentheses (`(`, `)`) or brackets (`[`, `]`), break lines before operands not after, align multiple arguments on the next line under the previous argument.  Indent blocks. Minimize vertical spaces.  Assume the reader of your program knows Stan---don't document the language. Use functions with descriptive names (verbs) wherever possible.

- *Keep Stan code in standalone files.*  This makes it easier to index line numbers in parser or runtime errors and easier to share programs among different clients and the users' group.

- *To debug, build a program up from simpler tested parts in steps and validate on simulated data.*  Do not start trying to implement a long program and then provide it real data.  Instead, begin with simple programs tested on simulated data (for posterior interval coverage, not for point estimate error), then continue to refine the program and data simulator until it is ready for real data.  








