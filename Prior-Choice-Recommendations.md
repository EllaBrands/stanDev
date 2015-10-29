* Don't use uniform priors, or hard constraints more generally, unless the bounds represent true constraints (such as scale parameters being restricted to be positive, or correlations restricted to being between -1 and 1).  Some examples:
  * You think a parameter could be anywhere from 0 to 1, so you set the prior to uniform(0,1).  Try normal(.5,.5) instead.
  * A scale parameter is restricted to be positive and you want to give it a vague prior, so you set to uniform(0,100) (or, worse, uniform(0,1000)).  If you just want to be vague, you could just specify no prior at all, which in Stan is equivalent to a noninformative uniform prior on the parameter.  Or you could give a weak prior such as exponential with expected value 10 (that's exponential(0.1), I think) or half-normal(0,10) (that's implmented as normal(0,10) with a <lower=0> constraint in the declaration of the parameter) or half-Cauchy(0,5) or even something more informative such as half-normal(0,1) or half-t(3,0,1).

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

* Prior for linear regression
  * Rescale predictors and outcomes
  * By default, use the same sorts of priors we recommend for logistic regression?

* Prior for the regression coefficients in logistic regression (non-sparse case)

  A recommended weakly informative prior
  * beta ~ student_t(nu,0,s)
    where s is chosen to provide weak information on the expected
    scale, and 3<nu<7.

    Assuming that nonbinary variables have been scaled to have mean 0  and standard deviation 0.5, Gelman et al (2008) (A Weakly Informative Default Prior Distribution for Logistic and Other
    Regression Models) recommended student_t(1,0,2.5) (Cauchy distribution). Later it has been observed that this has too thick tails, so that in cases where data is not informative (e.g., in case of separation) the sampling from the posterior is challenging (see, e.g., Ghosh et al, 2015,
    http://arxiv.org/abs/1507.07170). Thus Student's t distribution with higher degrees of freedom is recommended. There is not yet conclusive results what specific value should be recommended, and thus the current recommendation is to choose 3<nu<7. Normal distribution is not recommended as a weakly informative prior, because it is not robust (see, O'Hagan (1979) On outlier rejection phenomena in Bayes inference.). Normal distribution would be fine as an informative prior.

* Sparsity promoting prior for the regression coefficients

  See Vehtari & Piironen (2015). Projection predictive variable selection using Stan+R. [arXiv:1508.02502](http://arxiv.org/abs/1508.02502)

* Degrees of freedom in Student's t distribution

  A recommended easy default option is
  * nu ~ gamma(2,0.1); 

  This was proposed and anlysed by Juárez and Steel (2010) (Model-based clustering of non-Gaussian panel data based on skew-t distributions. Journal of Business & Economic Statistics 28, 52–66.). Juárez and Steel compere this to Jeffreys prior and report that the difference is small.  Simpson et al (2014) (arXiv:1403.4630) propose a theoretically well justified "penalised complexity (PC) prior", which they show to have a good behavior for the degrees of freedom, too. PC prior might be the best choice, but requires numerical computation of the prior (which could computed in a grid and interpolated etc.). It would be feasible to implement it in Stan, but it would require some work. Unfortunately no-one has compared PC prior and this gamma prior directly, but based on the discussion with Daniel Simpson, although PC prior would be better this gamma(2,0.1) prior is not a bad choice. Thus, I would use it until someone implements the PC prior for degrees of freedom of the Student's t in Stan.

* Elasticities (regressions on log-log scale)
  * We expect these to be between 0 and 1. But we don't want hard constraints.  So maybe N(.5,.5) is a good default

* A single correlation parameter
  * Uniform(-1,1) is noninformative
  * Other times we will expect a correlation to be positive, for example the correlation between a pre-test and post-test.  Here it could make sense to model using some latent score, that is to move to some sort of IRT model.  We should give an example of this for the wiki

* Covariance matrix
  * Ben recommends LKJ(4).  LKJ(1) is uniform on the correlation matrix but this gets weird if you look at the marginals.  Ben thinks 4 df is a reasonable default.