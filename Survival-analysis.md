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

