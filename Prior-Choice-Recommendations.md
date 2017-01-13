* General principles:
  * [Simpson et al. paper](http://arxiv.org/abs/1403.4630) has principles based on nestable models
  * Many times when people recommend default priors, they are restricting to some version of conjugacy for closed forms or for Gibbs; we don't care about that
  * Computational goal in Stan:  reducing instability which can typically arise from bad geometry in the posterior, heavy tails that can cause Stan to adapt poorly and have heavy tails
  * Some principles we don't like:  invariance, Jeffreys, entropy
  * Weakly informative prior should contain enough information to regularize:  the idea is that the prior rules out unreasonable parameter values but is not so strong as to rule out values that might make sense
  * Weakly informative rather than fully informative:  the idea is that the loss in precision by making the prior a bit too weak (compared to the true population distribution of parameters or the current expert state of knowledge) is less serious than the gain in robustness by including parts of parameter space that might be relevant.  It's been hard for us to formalize this idea.

* Don't use uniform priors, or hard constraints more generally, unless the bounds represent true constraints (such as scale parameters being restricted to be positive, or correlations restricted to being between -1 and 1).  Some examples:
  * You think a parameter could be anywhere from 0 to 1, so you set the prior to uniform(0,1).  Try normal(.5,.5) instead.
  * A scale parameter is restricted to be positive and you want to give it a vague prior, so you set to uniform(0,100) (or, worse, uniform(0,1000)).  If you just want to be vague, you could just specify no prior at all, which in Stan is equivalent to a noninformative uniform prior on the parameter.  Or you could give a weak prior such as exponential with expected value 10 (that's exponential(0.1), I think) or half-normal(0,10) (that's implmented as normal(0,10) with a <lower=0> constraint in the declaration of the parameter) or half-Cauchy(0,5) or even something more informative such as half-normal(0,1) or half-t(3,0,1).

* Super-weak priors to better diagnose major problems
  * For example, normal(0,100) priors added for literally every parameter in the model.  The assumption is that everything's on unit scale so these priors will have no effect--unless the model is blowing up from nonidentifiablity.  The super-weak prior allows you to see problems without the model actually blowing up.  We could even have a setting in Stan that automatically adds this information for every parameter.

* Super-constraining priors; you should also specify inits
  * Consider the following scenario:  You fit a model, and in order to keep your inference under control, you set some of the parameters to fixed, preset values.  Now you want to let these parameters float; that is, you want to estimate them from data.  But if you just jump all the way to flat priors, or even weakly informative priors, your inferences blow up, as there are still things you need to understand about your model.  So you ease into it by giving your parameters very strong priors.  For example, if you had a parameter that you'd given a preset value of 4, you try it with a normal (4, 0.1) prior, or maybe normal (4, 1).

    When you do this, you also should also specify initial values.  Why?  Because, by default, Stan draws inits uniformly from (-1, 1) (or maybe it's (-2, 2); I don't remember).  If you have a parameter that you want to set to be near 4, say, you should set inits to be near 4 also.

* Generic prior for anything
  * Andrew has been using independent N(0,1), as in section 3 of this paper:  http://www.stat.columbia.edu/~gelman/research/published/stan_jebs_2.pdf
    * I don't mind the short tails; if you think they're a problem because you think a parameter could be far from 0, that's information that can and should be included in the prior
    * I don't mind the prior independence as long as you've thought about combining or transforming parameters appropriately (see below)
  * Aki prefers student_t(3,0,1), something about some shape of some curve, he put it on the blackboard and I can't remember

  * Prior will depend on parameterization
  * Reparameterize to aim for approx prior independence (examples in Gelman, Bois, Jiang, 1996).  Simple example is to move from (theta_1, theta_2) to (theta_1 + theta_2, theta_1 - theta_2) if that makes sense in the context of the model.
  * Aim to keep all parameters scale-free.  Many ways of doing this:
    * Scale by sd of data.  This is done in education settings, here the sd is the sd of test scores of all kids in a single grade, for example
    * In a regression, take logs of (positive-constrained) predictors and outcomes, then coefs can be interpreted as elasticities
    * Scale by some conventional value, for example if a parameter has a "typical" value of 4.5, you could work with log(theta/4.5).  We did some things like this in our PK/PD project with Sebastian1. For example, in epidemiological studies it is common to standardize with the expected number of events.
  * Once parameters are scale-free, we want them to be on "unit scale"--that is, of order of magnitude 1.  We don't want parameters to have values like 0.01 or 100, we want them to be not too far or too close to 0
    * But sometimes parameters really are close to 0 on a real scale, and we need to allow that.  For example, the tiny effect of some ineffective treatment.  We would not want to "artificially" scale this up to 1 just to follow some principle.  Here's an example:  in education it's hard to see big effects.  An effect of .1 sd is actually pretty damn big, given that "1 sd" represents all the variation across kids.  In a setting where true effects are small, we need to allow that.

* Generic weakly informative prior
  * This talk from 2014:  http://www.stat.columbia.edu/~gelman/presentations/wipnew2_handout.pdf (slightly updated version of 2011 paper)
  * One principle:  write down what you think the prior should be, then spread it out.  The idea is that the cost of setting the prior too narrow is more severe than the cost of setting it too wide.  I've been having trouble formalizing this idea. 
  * See this paper:  http://arxiv.org/pdf/1403.4630.pdf , I don't fully understand it all but it seems like a step in the right direction

* Setting priors based on the prior predictive distribution (I. J. Good:  "Method of imaginary results")
  * Example:  LKJ etc, setting priors on cov matrices and looking at the implied prior on correlations
  * Example:  in linear regression, setting a prior on R-squared.  Ben came up with this idea and implemented it in stan_lm() in rstanarm)
  * Example:  "On the Hyperprior Choice for the Global Shrinkage Parameter in the Horseshoe Prior" by Juho Piironen and Aki Vehtari.  https://arxiv.org/abs/1610.05559  Not just horseshoe, also hierarchical shrinkage model

* Boundary-avoiding priors for modal estimation (posterior mode, MAP, marginal posterior mode, marginal maximum likelihood, MML)
  * These are for parameters such as group-level scale parameters, group-level correlations, group-level covariance matrix
  * What all these parameters have in common is that (a) they're defined on a space with a boundary, and (b) the likelihood, or marginal likelihood, can have a mode on the boundary. Most famous example is the group-level scale parameter tau for the 8-schools hierarchical model.
  * With full Bayes the boundary shouldn't be a problem (as long as you have any proper prior).
  * But with modal estimation, the estimate can be on the boundary, which can create problems in posterior predictions.  For example, consider a varying-intercept varying-slope multilevel model which has an intercept and slope for each group.  Suppose you fit marginal maximum likelihood and get a modal estimate of 1 for the group-level correlation.  Then in your predictions the intercept and slope will be perfectly correlated, which in general will be unrealistic.
  * For a one-dimensional parameter restricted to be positive (e.g., the scale parameter in a hierarchical model), we recommend Gamma(2,0) prior (that is, p(tau) proportional to tau) which will keep the mode away from 0 but still allows it to be arbitrarily close to the data if that is what the likelihood wants.  For details see this paper by Chung et al.:  http://www.stat.columbia.edu/~gelman/research/published/chung_etal_Pmetrika2013.pdf
    * Gamma(2,0) biases the estimate upward.  When number of groups is small, try Gamma(2,1/A), where A is a scale parameter representing how high tau can be.
  * For a correlation parameter, a Beta(2,2) parameter on 2*(rho - 1/2) will keep the point estimate away from the boundary.
    * Again, for full Bayes, a uniform prior on rho will serve a similar purpose.  But for modal estimation, the Beta(2,2) prior will keep the estimate off the boundary while allowing it to be arbitrarily close if so demanded by the data.
  * For a hierarchical covariance matrix, we suggest a Wishart (not inverse-Wishart) prior; see this paper by Chung et al.:  http://www.stat.columbia.edu/~gelman/research/published/chung_cov_matrices.pdf
  * For _non_ hierarchical variance and covariance parameters--that is, when these are estimated using direct data--we don't usually need these boundary-avoiding priors, even for modal estimation, because the likelihood goes to zero at the degenerate points of the model.  Hierarchical models are different because the data for these distributions are not directly observed, and it turns out that zero group-level variances can be consistent with the data.
  * Thus, boundary-avoiding priors for modal estimation can be useful for other models with indirect data, such as latent-parameter models and measurement-error models.

* Prior for linear regression
  * Rescale predictors and outcomes
  * By default, use the same sorts of priors we recommend for logistic regression?

* Prior for the regression coefficients in logistic regression (non-sparse case)

  A recommended weakly informative prior
  * beta ~ student_t(nu,0,s)
    where s is chosen to provide weak information on the expected
    scale, and 3<nu<7.

    Assuming that nonbinary variables have been scaled to have mean 0  and standard deviation 0.5, Gelman et al (2008) (A Weakly Informative Default Prior Distribution for Logistic and Other Regression Models) recommended student_t(1,0,2.5) (Cauchy distribution). Later it has been observed that this has too thick tails, so that in cases where data is not informative (e.g., in case of separation) the sampling from the posterior is challenging (see, e.g., Ghosh et al, 2015, http://arxiv.org/abs/1507.07170). Thus Student's t distribution with higher degrees of freedom is recommended. There is not yet conclusive results what specific value should be recommended, and thus the current recommendation is to choose 3<nu<7. Normal distribution is not recommended as a weakly informative prior, because it is not robust (see, O'Hagan (1979) On outlier rejection phenomena in Bayes inference.). Normal distribution would be fine as an informative prior.

* Sparsity promoting prior for the regression coefficients ("Bayesian model reduction")

  See Piironen and Vehtari (2015). Projection predictive variable selection using Stan+R. [arXiv:1508.02502](http://arxiv.org/abs/1508.02502)
  
  Also "On the Hyperprior Choice for the Global Shrinkage Parameter in the Horseshoe Prior" by Juho Piironen and Aki Vehtari.  https://arxiv.org/abs/1610.05559 

  We want to compare this horseshoe or HS implementation in rstarm to lasso and glmnet

* Prior for degrees of freedom in Student's t distribution

  A recommended easy default option is
  * nu ~ gamma(2,0.1); 

  This was proposed and anlysed by Juárez and Steel (2010) (Model-based clustering of non-Gaussian panel data based on skew-t distributions. Journal of Business & Economic Statistics 28, 52–66.). Juárez and Steel compare this to Jeffreys prior and report that the difference is small.  Simpson et al (2014) (arXiv:1403.4630) propose a theoretically well justified "penalised complexity (PC) prior", which they show to have a good behavior for the degrees of freedom, too. PC prior might be the best choice, but requires numerical computation of the prior (which could computed in a grid and interpolated etc.). It is available as **INLA:::inla.pc.ddof** for dof>2 and a standardized Student's-t. It would be feasible to implement it in Stan, but it would require some work. Unfortunately no-one has compared PC prior and this gamma prior directly, but based on the discussion with Daniel Simpson, although PC prior would be better this gamma(2,0.1) prior is not a bad choice. Thus, I would use it until someone implements the PC prior for degrees of freedom of the Student's t in Stan.

* Prior for elasticities (regressions on log-log scale)
  * We expect these to be between 0 and 1. But we don't want hard constraints.  So maybe N(.5,.5) is a good default

* Prior for a single correlation parameter
  * Uniform(-1,1) is noninformative
  * Other times we will expect a correlation to be positive, for example the correlation between a pre-test and post-test.  Here it could make sense to model using some latent score, that is to move to some sort of IRT model.  We should give an example of this for the wiki
  * If doing modal estimation, see section on Boundary Avoiding Priors above
  * [Penalised complexity priors are available](http://arxiv.org/abs/1608.08941) for auto-regressive processes. For order=1, different priors are recommended depending on the natural base model. 

* Prior for a covariance matrix
  * Ben recommends LKJ(4).  LKJ(1) is uniform on the correlation matrix but this gets weird if you look at the marginals.  Ben thinks 4 df is a reasonable default. No, Ben doesn't (and it is a shape parameter rather than the degrees of freedom). Anything over 2 is very concentrated at the identity matrix.
  * If doing modal estimation, see section on Boundary Avoiding Priors above

* Prior for scale parameters in hierarchical models
  * Gelman (2006) suggested half-Cauchy with mode at 0 and scale set to a large value (in the 8-schools example, we used the value 25), or with the scale estimated from data in a hierarchical-hierarchical setting in which there are many variance parameters which can be given a common prior.
  * The Gelman (2006) recommendations may be too weak for many purposes.  If the number of groups is small, the data don't provide much information on the group-level variance, and so it can make sense to use stronger prior information, in two ways.  First:  Cauchy might be too broad, maybe better to use something like a t_4 or even half-normal if you don't think there's a chance of any really big values.  Second:  maybe the scale parameter for this hyperprior should be set to something reasonable, not to something large.  This would suggest something like half-normal(0,1) or half-t(4,0,1) as default choices.
  * Historically, a prior on the scale parameter with a long right tail has been considered "conservative" in that it allows for large values of the scale parameter which in turn correspond to minimal pooling.  But from a modern point of view, minimal pooling is not a default, and a statistical method that underpools can be thought of as overreacting to noise and thus "anti-conservative."
  * If doing modal estimation, see section on Boundary Avoiding Priors above

* Prior for cutpoints in ordered logit or probit regression
  * For full Bayes, uniform priors typically should be ok, I think.
  * For modal estimation, put in some pseudodata in each category to prevent "cutpoint collapse."
  * Cutpoints are ordered (by definition).  Put the prior on the differences between the cutpoints rather than the cutpoints themselves.
  * Alternatively, put a prior on the cutpoints and partially pool them, not to a constant, but to a linear function.  That is, if the cutpoints are c[1],...,c[K], don't do c[k] ~ normal(mu, sigma); instead do c[k] ~ normal(a + b*k, sigma), estimating the hyperparameters a and b from data.  It will probably make sense to put informative priors on a, b, and sigma too.
  * Need to flesh out this section with examples.

* Priors for Gaussian processes
  * If the range of the x-axis values are pre-scaled to unit range (0-1, -.5-.5, etc), using the "inverse-lengthscale" parameterization (`CovMat[i,j] <- exp(dx*inv_rho) * eta ;`) means that for values of the inverse-lengthscale parameter ('inv_rho') between 0 & 1 implies a covariance structure from which sampled functions will be essentially linear (at least monotonic) across the range of the data. Thus, combined with a lower bound on 'inv_rho' at zero, the typical peaked-at-zero priors like 'inv_rho ~ normal(0,1)' will reflect a bias for simpler functions. Using a peaked-at-zero-but-heavy-tailed prior like 'inv_rho ~ student_t(4,0,1)' gives more credibility to more wiggly functions, but maintains the overall bias for simplicity.
  * If the range of the y-axis are pre-scaled to unit range, a prior of `normal(0,1)` on both the "signal magnitude" parameter `eta` and the "noise magnitude" parameter `sigma` reflects a prior on the signal-to-noise ratio that is peaked at .5.
  * For Matern fields, then the [joint penalised complexity prior](http://arxiv.org/abs/1503.00256) is available for the parameters (variance, range) parameters
  * In practice when we fit Gaussian processes we often either set the length-scale parameter to a fixed value corresponding to some desired level of smoothing, or we give it a strong prior.  The identification issues are real, and have to do with things such as, it's hard to identify much going on at a lower wavelength than the data spacing or a higher wavelength than the total range of the data, hence it can make perfect sense to either fix the value of hyperparameters or give them strong priors to make the model do what you want it to do.

* When using informative priors, be explicit about every choice; write a sentence about each parameter in the model.
  * For an example see section 4.1 of this paper:  http://www.stat.columbia.edu/~gelman/research/unpublished/objectivityr3.pdf