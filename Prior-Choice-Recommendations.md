# 5 levels of priors
  * Flat prior (not usually recommended);
  * Super-vague but proper prior:  normal(0, 1e6) (not usually recommended);
  * Weakly informative prior, very weak:  normal(0, 10);
  * Generic weakly informative prior:  normal(0, 1);
  * Specific informative prior:  normal(0.4, 0.2) or whatever.  Sometimes this can be expressed as a scaling followed by a generic prior:  theta = 0.4 + 0.2*z; z ~ normal(0, 1);

The above numbers assume that parameters are roughly on unit scale, as is done in education (where 0 is average test score in some standard population (e.g., all students at a certain grade level) and 1 is sd of test scores in that population) or medicine (where 0 is zero dose and 1 is a standard dose such as 10mcg/day of cyanocobalamin, 1,000 IU/day cholecalciferol, etc.; these examples come from Sander Greenland).

In addition, statements such as "informative" or "weakly informative" depend crucially on what questions are being asked (a point that is related to the idea that the prior can often only be understood in the context of the likelihood (http://www.stat.columbia.edu/~gelman/research/published/entropy-19-00555-v2.pdf)).

Flat and super-vague priors are not usually recommended and some thought should included to have at least weakly informative priors. For example, it is common to expect realistic effect sizes to be of order of magnitude 0.1 on a standardized scale (for example, an educational innovation that might improve test scores by 0.1 standard deviations).  In that case, a prior of N(0,1) could be considered very informative, in a bad way, in that it puts most of its mass on parameter values that are unrealistically large in absolute value.  The general point here is that if consider a prior to be "weak" or "strong," this is a property not just of the prior but also of the question being asked.

When we say this prior is "weakly informative," what we mean is that, if there's a reasonably large amount of data, the likelihood will dominate, and the prior will not be important.  If the data are weak, though, this "weakly informative prior" will strongly influence the posterior inference.  The phrase "weakly informative" is implicitly in comparison to a default flat prior. If there is no prior information directly in scale of parameters, it is common to have some information on the scale for the order of magnitude of the outcomes which can be used to make weakly informative priors ([Gabry, Simpson, Vehtari, Betancourt, and Gelman, 2019](https://doi.org/10.1111/rssa.12378)).

# Independence

We commonly set up our models so that parameters are independent in their prior distributions.  This is partly for convenience and partly because setting up the model in this way is more understandable.  But this means that we have to be careful with parameterization.  For an example of a parameterization set up so that prior independence seems like a reasonable assumption, see section 2.2 of this paper:  http://www.stat.columbia.edu/~gelman/research/published/bois2.pdf

With hierarchical models, it can be possible to check prior independence using a posterior predictive check.  (The check is posterior given the data but it is prior in the sense of studying the distribution of parameters across groups).  Section 3.3 of this paper, http://www.stat.columbia.edu/~gelman/research/published/bois2.pdf, gives an example of a check in which we could strongly reject the model of prior independence.  The result of this check motivated us to expand our model; prior independence seemed like a reasonable assumption in this expanded model, and it was also consistent with the data.  So, in this example, the appropriate solution was not to allow large prior correlations but rather to set up the model in a way that made substantive sense.

For a general discussion of the point that reparameterization is particularly relevant to Bayesian inference, because of the usual default assumption of prior independence, see section 5.1 of this paper:  http://www.stat.columbia.edu/~gelman/research/published/parameterization.pdf

Even when we explicitly model prior dependence (so we are _not_ assuming prior independence), we typically use a multivariate model such as the LKJ prior in which prior independence (a diagonal covariance matrix) is the baseline.

Another example of a reparameterization is the t(nu, mu, sigma) distribution.  Here, we prefer to set up the prior in terms of nu, mu, sigma/(nu-2) or something like that, to account for the fact that the scale of the distribution (as measured by the sd or median absolute deviation) depends on nu as well as sigma.

For an example of a problem with the naive assumption of prior independence, see section 2.3 of this paper:  Sensitivity Analysis, Monte Carlo Risk Analysis, and Bayesian Uncertainty Assessment, by Sander Greenland, Risk Analysis, 21, 579-583 (2001).  Also related are the papers, 
Putting Background Information About Relative Risks into Conjugate Prior Distributions, by Sander Greenland, Biometrics 57, 663-670 (2001), Simpson’s Paradox From Adding Constants in Contingency Tables as an Example of Bayesian Noncollapsibility, by Sander Greenland, American Statistician 64, 340-344 (2010).

# General principles
  * [Simpson et al. paper](http://arxiv.org/abs/1403.4630) has principles based on nestable models
  * Many times when people recommend default priors, they are restricting to some version of conjugacy for closed forms or for Gibbs; we don't care about that
  * Computational goal in Stan:  reducing instability which can typically arise from bad geometry in the posterior, heavy tails that can cause Stan to adapt poorly and have heavy tails
  * Some principles we don't like:  invariance (that is, setting up a prior based on invariance principles without a sense of what is the meaning of the parameter), Jeffreys (see BDA for brief discussion of this point), maximum entropy
  * Weakly informative prior should contain enough information to regularize:  the idea is that the prior rules out unreasonable parameter values but is not so strong as to rule out values that might make sense
  * Weakly informative rather than fully informative:  the idea is that the loss in precision by making the prior a bit too weak (compared to the true population distribution of parameters or the current expert state of knowledge) is less serious than the gain in robustness by including parts of parameter space that might be relevant.  It's been hard for us to formalize this idea.
  * When using informative priors, be explicit about every choice; write a sentence about each parameter in the model.
    * For an example see section 4.1 of this paper: http://www.stat.columbia.edu/~gelman/research/unpublished/objectivityr3.pdf


* Don't use uniform priors, or hard constraints more generally, unless the bounds represent true constraints (such as scale parameters being restricted to be positive, or correlations restricted to being between -1 and 1).  Some examples:
  * You think a parameter could be anywhere from 0 to 1, so you set the prior to uniform(0,1).  Try normal(.5,.5) instead.
  * A scale parameter is restricted to be positive and you want to give it a vague prior, so you set to uniform(0,100) (or, worse, uniform(0,1000)).  If you just want to be vague, you could just specify no prior at all, which in Stan is equivalent to a noninformative uniform prior on the parameter.  Or you could give a weak prior such as exponential with expected value 10 (that's exponential(0.1), I think) or half-normal(0,10) (that's implemented as normal(0,10) with a <lower=0> constraint in the declaration of the parameter) or half-Cauchy(0,5) or even something more informative such as half-normal(0,1) or half-t(3,0,1).

* Super-weak priors to better diagnose major problems
  * For example, normal(0,100) priors added for literally every parameter in the model.  The assumption is that everything's on unit scale so these priors will have no effect--unless the model is blowing up from nonidentifiablity.  The super-weak prior allows you to see problems without the model actually blowing up.  We could even have a setting in Stan that automatically adds this information for every parameter.

* Super-constraining priors; you should also specify inits
  * Consider the following scenario:  You fit a model, and in order to keep your inference under control, you set some of the parameters to fixed, preset values.  Now you want to let these parameters float; that is, you want to estimate them from data.  But if you just jump all the way to flat priors, or even weakly informative priors, your inferences blow up, as there are still things you need to understand about your model.  So you ease into it by giving your parameters very strong priors.  For example, if you had a parameter that you'd given a preset value of 4, you try it with a normal (4, 0.1) prior, or maybe normal (4, 1).

    When you do this, you also should also specify initial values.  Why?  Because, by default, Stan draws inits uniformly from (-1, 1) (or maybe it's (-2, 2); I don't remember).  If you have a parameter that you want to set to be near 4, say, you should set inits to be near 4 also.

# How informative is the prior?

* "The prior can often only be understood in the context of the likelihood": http://www.stat.columbia.edu/~gelman/research/published/entropy-19-00555-v2.pdf
Or, more generally, in the context of the estimating function, or of the information in the data.

* Prior predictive checking helps to examine how informative the prior on parameters is in the scale of the outcome: https://doi.org/10.1111/rssa.12378

* Here's an idea for not getting tripped up with default priors:  For each parameter (or other qoi), compare the posterior sd to the prior sd.  If the posterior sd for any parameter (or qoi) is _more than 0.1 times the prior sd_, then print out a note:  "The prior distribution for this parameter is informative." Then the user can go back and check that the default prior makes sense for this particular example.

Michael Betancourt has some related ideas here:  https://betanalpha.github.io/assets/case_studies/principled_bayesian_workflow.html

# Default prior for treatment effects scaled based on the standard error of the estimate

Erik van Zwet suggests an Edlin factor of 1/2.  Assuming that the existing or published estimate is unbiased with known standard error, this corresponds to a default prior that is normal with mean 0 and sd equal to the standard error of the data estimate.  This can't be right--for any given experiment, as you add data, the standard error should decline, so this would suggest that the prior depends on sample size.  (On the other hand, the prior can often only be understood in the context of the likelihood; http://www.stat.columbia.edu/~gelman/research/published/entropy-19-00555-v2.pdf, so we can't rule out an improper or data-dependent prior out of hand.)

Anyway, the discussion with Zwet got me thinking.  If I see an estimate that's 1 se from 0, I tend not to take it seriously; I partially pool it toward 0.  So if the data estimate is 1 se from 0, then, sure, the normal(0, se) prior seems reasonable as it pools the estimate halfway to 0.  But if the data estimate is, say, 4 se's from zero, I wouldn't want to pool it halfway:  at this point, zero is not so relevant.  This suggests something like a t prior.  Again, though, the big idea here is to scale the prior based on the standard error of the estimate.

Another way of looking at this prior is as a formalization of what we do when we see estimates of treatment effects.  If the estimate is only 1 standard error away from zero, we don't take it too seriously:  sure, we take it as some evidence of a positive effect, but far from conclusive evidence--we partially pool it toward zero.  If the estimate is 2 standard errors away from zero, we still think the estimate has a bit of luck to it--just think of the way in which researchers, when their estimate is 2 se's from zero, (a) get excited and (b) want to stop the experiment right there so as not to lose the magic--hence some partial pooling toward zero is still in order.  And if the estimate is 4 se's from zero, we just tend to take it as is.


# Generic prior for anything
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
  * Once parameters are scale-free, we want them to be on "unit scale"--that is, of order of magnitude 1.  We don't want parameters to have values like 0.01 or 100, we want them to be not too far or too close to 0.  How should we do this?  The typical method is to divide by the scale. We prefer a robust estimator of the scale (such as the MAD) over the sample standard deviation. 
    * But sometimes parameters really are close to 0 on a real scale, and we need to allow that.  For example, the tiny effect of some ineffective treatment.  We would not want to "artificially" scale this up to 1 just to follow some principle.  Here's an example:  in education it's hard to see big effects.  An effect of .1 sd is actually pretty damn big, given that "1 sd" represents all the variation across kids.  In a setting where true effects are small, we need to allow that.

# Story: When the generic prior fails. The case of the Negative Binomial.

The generic prior for everything can fail dramatically when the parameterization of the distribution is bad. One example that pops up from time to time (both in INLA and rstanarm) is the problems in putting priors on the over-dispersion parameter of the negative binomial distribution. 

The `neg_binomial_2` distribution in Stan is parameterized so that the mean is `mu` and the variance is `mu*(1 + mu/phi)`. If you use the "generic prior for everything" for phi, such as a `phi ~ half-N(0,1)`, then most of the prior mass is on models with a large amount of over-dispersion. This will lead to a prior-data conflict if the data only exhibits a small amount of over-dispersion. 

The generic prior works much much better on the parameter `1/phi`.  Even better, you can use `1/sqrt(phi)`. This can be motivated by noticing you can write a negative binomial a `y | g ~ Poisson(g*mu)`, `g ~ gamma(phi,phi)` and the standard deviation of `g` is `1/sqrt(phi)`.  Some more information is in the second-last section of [this blog](http://andrewgelman.com/2018/04/03/justify-my-love/).

# Generic weakly informative prior
  * This talk from 2014:  http://www.stat.columbia.edu/~gelman/presentations/wipnew2_handout.pdf (slightly updated version of 2011 paper)
  * One principle:  write down what you think the prior should be, then spread it out.  The idea is that the cost of setting the prior too narrow is more severe than the cost of setting it too wide.  I've been having trouble formalizing this idea. 
  * See this paper:  http://arxiv.org/pdf/1403.4630.pdf , I don't fully understand it all but it seems like a step in the right direction

# Setting priors based on the prior predictive distribution (I. J. Good:  "Method of imaginary results")
  * Example:  LKJ etc, setting priors on cov matrices and looking at the implied prior on correlations
  * Example:  in linear regression, setting a prior on R-squared.  Ben came up with this idea and implemented it in stan_lm() in rstanarm)
  * Example:  "On the Hyperprior Choice for the Global Shrinkage Parameter in the Horseshoe Prior" by Juho Piironen and Aki Vehtari.  https://arxiv.org/abs/1610.05559  Not just horseshoe, also hierarchical shrinkage model

# Boundary-avoiding priors for modal estimation (posterior mode, MAP, marginal posterior mode, marginal maximum likelihood, MML)
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

# Prior for linear regression
  * Rescale predictors and outcomes
  * By default, use the same sorts of priors we recommend for logistic regression?

# Prior for the regression coefficients in logistic regression (non-sparse case)

  A recommended weakly informative prior
  * beta ~ student_t(nu,0,s)
    where s is chosen to provide weak information on the expected
    scale, and 3<nu<7.

    Assuming that nonbinary variables have been scaled to have mean 0  and standard deviation 0.5, Gelman et al (2008) (A Weakly Informative Default Prior Distribution for Logistic and Other Regression Models) recommended student_t(1,0,2.5) (Cauchy distribution). Later it has been observed that this has too thick tails, so that in cases where data is not informative (e.g., in case of separation) the sampling from the posterior is challenging (see, e.g., Ghosh et al, 2015, http://arxiv.org/abs/1507.07170). Thus Student's t distribution with higher degrees of freedom is recommended. There is not yet conclusive results what specific value should be recommended, and thus the current recommendation is to choose 3<nu<7. Normal distribution is not recommended as a weakly informative prior, because it is not robust (see, O'Hagan (1979) On outlier rejection phenomena in Bayes inference.). Normal distribution would be fine as an informative prior.

# Scaling

As noted above, we've moved away from the Cauchy and I (Andrew) am now using default normal(0.2.5) for rstanarm and normal(0,1) for my own work.  I think the right way to think about the Cauchy is that it's a normal with an unknown scale, so it could be appropriate if applied in a setting where the prior is unscaled and where we have no idea what are the scales of the problem at hand.  In a real life problem the user should have some sense of a scale, and in a completely blind problem (for example, you're running some sort of automatic feature detection program), you should be able to do some prescaling. 

# Data-dependent scaling

By default we scale the prior for regression coefficients based on the data. (see what is done in rstanarm and RAOS). I don't think there's any way around this.  We've been talking for awhile about formalizing this idea so as to move beyond the not-fully-Bayesian practice of data-dependent priors.  Aki writes:  "Instead of talking not-fully-Bayesian practice or double use of data, it might be better to say that we are doing 1+\epsilon use of data (1+\epsilon dipping?), and in most cases this can be done so that the benefit from stabilizing the inference overcomes the problems with 'uninformative' prior or prior whcih can be in bad conflict with the data."  "1 + epsilon dipping" . . . I like it!

# Sparsity promoting prior for the regression coefficients ("Bayesian model reduction")

  See Piironen and Vehtari (2015). Projection predictive variable selection using Stan+R. [arXiv:1508.02502](http://arxiv.org/abs/1508.02502)
  
  Also "On the Hyperprior Choice for the Global Shrinkage Parameter in the Horseshoe Prior" by Juho Piironen and Aki Vehtari.  https://arxiv.org/abs/1610.05559 

  We want to compare this horseshoe or HS implementation in rstanarm to lasso and glmnet

# Prior for degrees of freedom in Student's t distribution

  A recommended easy default option is
  * nu ~ gamma(2,0.1); 

  This was proposed and anlysed by Juárez and Steel (2010) (Model-based clustering of non-Gaussian panel data based on skew-t distributions. Journal of Business & Economic Statistics 28, 52–66.). Juárez and Steel compare this to Jeffreys prior and report that the difference is small.  Simpson et al (2014) (arXiv:1403.4630) propose a theoretically well justified "penalised complexity (PC) prior", which they show to have a good behavior for the degrees of freedom, too. PC prior might be the best choice, but requires numerical computation of the prior (which could computed in a grid and interpolated etc.). It is available as **INLA:::inla.pc.ddof** for dof>2 and a standardized Student's-t. It would be feasible to implement it in Stan, but it would require some work. Unfortunately no-one has compared PC prior and this gamma prior directly, but based on the discussion with Daniel Simpson, although PC prior would be better this gamma(2,0.1) prior is not a bad choice. Thus, I would use it until someone implements the PC prior for degrees of freedom of the Student's t in Stan.

  It's also a good idea to have a lower limit depending on how many finite moments for Student's t is desired. A reasonable default choice would be to declare
  * real\<lower=1\> nu;

  but in some cases it may be useful to have a higher lower limit.

# Prior for elasticities (regressions on log-log scale)
  * We expect these to be between 0 and 1. But we don't want hard constraints.  So maybe N(.5,.5) is a good default

# Prior for a single correlation parameter
  * Uniform(-1,1) is noninformative
  * Other times we will expect a correlation to be positive, for example the correlation between a pre-test and post-test.  Here it could make sense to model using some latent score, that is to move to some sort of IRT model.  We should give an example of this for the wiki
  * If doing modal estimation, see section on Boundary Avoiding Priors above
  * [Penalised complexity priors are available](http://arxiv.org/abs/1608.08941) for auto-regressive processes. For order=1, different priors are recommended depending on the natural base model. 

# Prior for a covariance matrix
  * There is a consensus now to decompose a covariance matrix into a correlation matrix and something else. There is less consensus on whether the something else should be standard deviations or variances and less consensus on what the prior should be.
  * If doing modal estimation, see section on [Boundary Avoiding Priors](#boundary-avoiding-priors-for-modal-estimation-posterior-mode-map-marginal-posterior-mode-marginal-maximum-likelihood-mml) above.

# Prior for scale parameters in hierarchical models
  * Gelman (2006) suggested half-Cauchy with mode at 0 and scale set to a large value (in the 8-schools example, we used the value 25), or with the scale estimated from data in a hierarchical-hierarchical setting in which there are many variance parameters which can be given a common prior.
  * The Gelman (2006) recommendations may be too weak for many purposes.  If the number of groups is small, the data don't provide much information on the group-level variance, and so it can make sense to use stronger prior information, in two ways.  First:  Cauchy might be too broad, maybe better to use something like a t_4 or even half-normal if you don't think there's a chance of any really big values.  Second:  maybe the scale parameter for this hyperprior should be set to something reasonable, not to something large.  This would suggest something like half-normal(0,1) or half-t(4,0,1) as default choices.
  * Historically, a prior on the scale parameter with a long right tail has been considered "conservative" in that it allows for large values of the scale parameter which in turn correspond to minimal pooling.  But from a modern point of view, minimal pooling is not a default, and a statistical method that underpools can be thought of as overreacting to noise and thus "anti-conservative."
  * If doing modal estimation, see section on Boundary Avoiding Priors above

# Prior for cutpoints in ordered logit or probit regression
  * For full Bayes, uniform priors typically should be ok, I think.
  * For modal estimation, put in some pseudodata in each category to prevent "cutpoint collapse."
  * Cutpoints are ordered (by definition).  Put the prior on the differences between the cutpoints rather than the cutpoints themselves.
  * Alternatively, put a prior on the cutpoints and partially pool them, not to a constant, but to a linear function.  That is, if the cutpoints are c[1],...,c[K], don't do c[k] ~ normal(mu, sigma); instead do c[k] ~ normal(a + b*k, sigma), estimating the hyperparameters a and b from data.  It will probably make sense to put informative priors on a, b, and sigma too.
  * Need to flesh out this section with examples.

# Priors for Gaussian processes
  * See recommendations in Section 18.3 in [Stan reference manual](https://github.com/stan-dev/stan/releases/download/v2.16.0/stan-reference-2.16.0.pdf) (especially the part titled "Priors for Gaussian Process Parameters")
  * For Matern fields, then the [joint penalised complexity prior](http://arxiv.org/abs/1503.00256) is available for the parameters (variance, range) parameters

# Priors for rstanarm
  * Default priors should all be autoscaled---this is particularly relevant for stan_glm().  In particular, for the normal-distribution link, prior_aux should be scaled to the residual sd of the data.  Currently it's an unscaled normal(0,5) which will be a very strong prior if the scale of the data happens to be large.
  * Should stan_lm() be tied to the R-squared prior?
  * If priors are user-specified, it seems to me that autoscaling should _not_ be the default.  Once you're specifying a prior from the outside, I'm guessing it will almost always be on the direct scale.

# Acknowledgements

We thank the U.S. Office of Naval Research for partial support of this work through grant N00014-15-1-2541, "Informative Priors for Bayesian Inference and Regularization."