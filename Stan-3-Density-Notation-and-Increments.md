Each probability distribution, such as Bernoulli or normal, has functions for its probability density function (PDF), cumulative distribution function (CDF), complementary cumulative distribution function (CCDF), inverse CDFs (ICDF), and pseudo random number generator (RNG)

## Function Suffixes

To distinguish which function for a distribution is being used and which scale.

#### Current Suffixes

scale      | PDF   | PMF     |  CDF     | CCDF     | inverse CDF | PRNG 
-----------|-------|---------|----------|----------|-------------|---------
*linear*   | n/a   |  n/a    | cdf      | ccdf     |    n/a      | rng   
*log*      | log   |  log    | cdf_log  | ccdf_log |    n/a      | n/a

#### Proposed Suffixes

scale    | PDF   | PMF  |  CDF  | CCDF  | diff of CDFs | inv CDF  | PRNG
---------|-------|------|-------|-------|--------------|----------|-----
*linear* | n/a   | n/a  | n/a   | n/a   | n/a          | inv_cdf  | rng
*log*    | lpdf  | lpmf | lcdf  | lccdf | ldiff_cdf    | n/a      | n/a     

Deprecate (not eliminate) existing functions.

##### Discussion

* The `pmf` vs. `pdf` suffix was up in the air after our last discussion.

* For inverse CDF, is the argument in (0, 1), or is it log odds or log prob?  Or do we have multiple parameterizations?

* Andrew suggests we should only supply the log functions where relevant and drop the "l" suffix and "pdf" and "pmf" suffixes, so that's

scale   | PDF   | PMF  |  CDF  | CCDF  | diff of CDFs
--------|-------|------|-------|-------|--------------
*log*   |  ε    |  ε   |   cdf |  ccdf | diff_cdf

where `ε` is the empty string, and then

scale    | RNG | inv CDF
---------|-----|--------
*linear* | rng | inv_cdf



## Vertical Bar Notation

To distinguish conditional densities from joint densities in the notation.

#### Current

`normal_log(y, mu, sigma)`

#### Proposed

`normal_lpdf(y | mu, sigma)`

and for multivariate outcomes, we could even write

`foo_lpdf(y1, y2 | alpha, beta)`         

#### Discussion

* Unconventional
    * not found in any other programming language
    * users may infer some kind of sampling rather than evaluation

* Adds one more horizontal space over `,`

## Normalization Control

The current difference between sampling notation and functions is confusing to everyone and it would be nice to be able to control it.

#### Current 

Sampling statement does not normalize:
```
y ~ normal(mu, sigma);
```

Function call does normalize:
```
increment_log_prob(normal_log(y, mu, sigma));
```

#### Proposed

PDF has special arguments for normalization, defaulting to `false`:

```
cauchy_lpdf<norm>(y | mu, tau);   // normalized
cauchy_lpdf(y | mu, tau);         // unnormalized
```

#### Discussion

* Ideally, everything would be normalized everywhere, but it's a computational burden

* The original proposal was for
    * `cauchy_lpdf<norm=true>(y | mu, tau)` : normalized
    * `cauchy_lpdf<norm=false>(y | mu, tau)` : unnormalized
    * `cauchy_lpdf(y | mu, tau)` : undecided

* Using just `cauchy_lpdf<norm>` rather than `cauchy_lpdf<norm=true>` is suboptimal if we want other arguments with values.

* The default is *unnormalized*
    * this is going to lead to a lot of confusion from users who expect normalization as the default
    * it's going to lead to confusion for users about where normalization is needed

* It would be great to only compute normalizing constants once on outside and use all normalized inside, but
    * won't work if there is branching on parameters
    * need the values on the inside for mixture models

* Aki suggests following Andrew's *BDA* notation with `q()` being unnormalized form of `p()`, so
    * instead of `beta_lpdf(y | a, b)` for the unnormalized pdf, write `beta_lqdf(y | a, b)`
    * would be confusing given `q` usage for quantiles in R
    

## Scalar/Vector Output Control

See:  https://github.com/stan-dev/stan/issues/1697

#### Current 

If y is vector (or array) sampling statement and function call compute sum of log densities
```
y ~ normal(mu, sigma);
sum_log_lik <- normal_log(y, mu, sigma)
```

In generative quantities, only the unvectorized form is alllowed.  
```
for (n in 1:N)
 log_lik[n] <- normal_log(y[n], mu, sigma)
sum_log_lik <- sum(log_lik);
```

#### Proposed

To be able to use vectorized form in generated quantities (and maybe sometimes elsewhere?) PDF has special arguments for vector output, defaulting to `false`:

```
log_lik <- normal_lpdf<norm=true,vector=true>(y | mu, tau);
```

or PDF has special arguments for summing, defaulting to `true`:

```
log_lik <- normal_lpdf<norm=true,sum=false>(y | mu, tau);
```

#### Discussion

Current form is difficult because use vectorized sampling statements, but not in generated quantities with RNGs or evaluation for WAIC, etc.

## Increment Log Density Statement

Make it clear that we're incrementing an underlying accumulator rather than forward sampling or defining a directed graph.

#### Current

1.  `increment_log_prob(normal_log(y, mu, sigma));`

1.  `y ~ normal(mu, sigma);`

#### Proposed

Replace both of these with a single version:

1.  `target += normal_lpdf(y | mu, sigma);`

where `target` is the target log density.

#### Discussion

* `target` is sufficiently neutral that nobody's going to analyze it

* need to make `target` a reserved word

* First consideration was to using streaming operator
    * `target << normal_lpdf(y | mu, sigma);`
    * too confusing because `<<` is unfamiliar and it's not really streaming

* Another consideration was to using streaming with nothing on left side
    * really confusing

* For consistency, we should 
    * allow `+=` for other variables
    * fallow `=` in place of `<-` and deprecate `<-`

* Do we allow users to access value of `target` even if they can't assign it?

* Could avoid streaming altogether and use a new symbol for the prefix like:
```
@normal_lpdf(mu | 0, 1);
@normal_lpdf(beta | mu, sigma);
@cauchy_lpdf(sigma | 0, 2.5);
```
* Do we allow vectors or other containers on the right-hand side, as we do for `increment_log_prob()` now?

## Link Functions in Density Names

Current approach doesn't scale.

#### Current

```
poisson_log(alpha) == poisson(exp(alpha))
bernoulli_logit(alpha) == bernoulli(inv_logit(alpha))
```

#### Proposed

```
poisson<link=log>(alpha)
```

#### Discussion

* Verbose.  What if there are two arguments, `link1` and `link2`?

* User's may overgeneralize from the general syntax to assume arguments that don't exist
    * so just use `bernoulli_logit` instead of `bernoulli<link=logit>`

## User-Defined Functions

#### Current

Use current versions of function names, with `_log` allowing sampling statement use and "_lp" allowing access to log density.

#### Proposed

Use proposed versions of suffixes, with `_pdf` supporting vertical bar notation.

Use "??" instead of "_l" in user-defined functions with access to increment-log-density streaming.

Deprecate (not eliminate) use of `_log` in sampling statements.

#### Discussion

* What do we use in place of `_lp` for functions that have access to `target`?  I don't think `_target` makes sense!