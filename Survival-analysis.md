# Survival analysis in Stan

Jonah Gabry http://discourse.mc-stan.org/t/survival-models-in-rstanarm/3998/28
> From the Stan team side I think we could do the following:
>
> - host the package in the stan-dev org on GitHub, give it a website on mc-stan.org (e.g. http://mc-stan.org/bayesplot/)
> - not do the majority of the coding but be involved enough to have input into the Stan implementations as well as the interface (to make sure it integrates nicely with the existing ecosystem).
> - ensure the package’s sustained existence on repositories like CRAN (particularly important in the case that core contributors to the package end their involvement, which does happen).
>
> As to whether this is a new package (maybe rstansurv?) or an addition to rstanarm (since rstanarm is already exactly what I described above except we wrote most of it), I’m not sure.
>
> I would suggest starting to put together a preliminary Stan program (maybe just start with one type of model and then add more later) as well as some R code for preparing data to pass to Stan, and then once there’s something to look at / try out I think it will be easier for us to see whether it makes sense to go with rstanarm or in a new package on stan-dev.

# Survival analysis models

- Accelerated failure time models (e.g. log-normal, log-logistic, Weibull) with censored data
- Cox-PH type models with a functional prior on baseline
- non-proportional models with interaction with time and covariates

# First implementations

## Cox-PH with I-splines for log cumulative baseline hazard (Royston & Parmar)

https://github.com/ermeel86/paramaetricsurvivalmodelsinstan

### References

- [Flexible parametric proportional-hazards and proportional-odds models for censored survival data, with application to prognostic modelling and estimation of treatment effects](https://www.ncbi.nlm.nih.gov/pubmed/12210632)
- [Monotone Regression Splines in Action](https://projecteuclid.org/euclid.ss/1177012761)
- [The current application of the Royston- Parmar model for prognostic modeling in health research: a scoping review](https://link.springer.com/content/pdf/10.1186%2Fs41512-018-0026-5.pdf)

### Remarks / Concerns

- The I-splines, in contrast to restricted cubic splines, guarantee monotonicity (in time) for the log cumulative hazard function, which strictly speaking is required for a consistent survival model.
- The original RP model used restricted cubic splines, which in particular are constrained to be linear beyond the boundary knots. This was advertised to be useful for out-of-sample prediction. It’s not clear, yet, how the I-isplines extrapolate beyond the boundary knots.

### Todos

- Try to reproduce further examples from the literature:
   - https://web.stanford.edu/~hastie/CASI_files/DATA/ncog.html
- add support for NAs
- Implement the proportional odds model version
- use survival formula parser for covariates
- implement time-varying hazard ratios, like in section 4.1. in R&P.
- Work with (smooth) random-walk priors for $\gamma$ in order to allow for more knots, c.f. http://mc-stan.org/users/documentation/case-studies/splines_in_stan.html

# Datasets for testing

- https://web.stanford.edu/%7Ehastie/CASI_files/DATA/ncog.html
- https://web.stanford.edu/~hastie/CASI/data.html
- https://github.com/chjackson/flexsurv-dev/blob/master/data/bc.txt
- http://stat.ethz.ch/R-manual/R-devel/library/MASS/html/leuk.html

## Cox-PH with integrated-B-splines for log cumulative baseline hazard (Royston & Parmar)

- *Link to GitHub follows*
- The R package `splines2` provides functionality to calculate integrated B-splines (hence the derivatives are simply B-splines)
- If all the weights for the integrated-B-splines-basis-functions are equal (and non-negative), the resulting function is linear with $0$ intercept. The corresponding linear combination of B-splines is a constant function. We could thus try and apply smooth random walk priors and allow for larger number of knots than usual.
