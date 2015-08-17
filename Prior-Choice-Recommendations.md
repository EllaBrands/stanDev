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