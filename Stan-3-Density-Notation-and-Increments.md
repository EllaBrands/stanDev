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

* For inverse CDF, we may want an alternative parameterization with a logit-scale parameter.

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

For pdfs and pmfs, we use the bar to separate the outcome from the parameters:

```
normal_lpdf(y | mu, sigma)
```

Ditto for cdfs, etc.

```
normal_lccdf(y | mu, sigma);
```

and for multivariate outcomes, we could even write

```
foo_lpdf(y1, y2 | alpha, beta)` 
```

#### Discussion

* Unconventional
    * not found in any other programming language
    * users may infer some kind of sampling rather than evaluation

* Not clear how we resolve pmf vs. pdf when it's mixed (e.g., a mixture of a pdf and pmf, as in a proper spike-and-slab)

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

The name of the function will get a `_norm` suffix before the `lpdf`, as in:

```
cauchy_norm_lpdf(y | mu, tau);    // normalized
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

Right now, the densities return the sum of log densities for vector inputs;  it'd be nice to be able to control this and produce a vector of outputs, which is necessary for WAIC and other similar calculations.

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
vector[N] y;
vector[N] ll;
...
ll <- vec_normal_norm_lpdf(y | mu, tau);
```

#### Discussion

* These suffixes are going to start piling up and nobody's going to be able to remember the order.

* Current form is difficult because use vectorized sampling statements, but not in generated quantities with RNGs or evaluation for WAIC, etc.

* Not clear we really are going to keep computing WAIC, etc., given the current focus on cross-validation.


## Increment Log Density Statement

Make it clear that we're incrementing an underlying accumulator rather than forward sampling or defining a directed graph.

#### Current

1.  `increment_log_prob(normal_log(y, mu, sigma));`

1.  `y ~ normal(mu, sigma);`

#### Proposed

Replace both of these with a single version:

1.  `target += normal_lpdf(y | mu, sigma);` where `target` is the target log density.

Furthermore,

1.  Deprecate `<-` for assignment and replace with `=`

1.  Allow general use of `+=` for incrementing variables.

1.  Do not allow general access to `target` --- it only lives on left of `+=`;  we supply an existing function to get its value.

1.  Make `target` a reserved word (minor backward-compatibility breaking)

1.  Do not allow vectors on the right of `+=` meaning to sum them all (currently, `increment_log_prob()` allows that (have to be careful here in our translator).

#### Discussion

* `target` is sufficiently neutral that nobody's going to overanalyze it

* First consideration was to using streaming operator
    * `target << normal_lpdf(y | mu, sigma);`
    * decided it was too confusing because `<<` is unfamiliar and it's not really streaming
    * `<< normal_lpdf(y | mu, sigma)` and just plain `normal_lpdf(y | mu, sigma)` also rejected

* Could avoid streaming altogether and use a new symbol for the prefix like:

```
@normal_lpdf(mu | 0, 1);
@normal_lpdf(beta | mu, sigma);
@cauchy_lpdf(sigma | 0, 2.5);
```

but nobody liked that idea other than me (Bob).

## Link Functions in Density Names

Current approach doesn't scale because we need to mark both link function (type of parameter) and name of function and its output.

#### Current

```
poisson_log(alpha) == poisson(exp(alpha))
bernoulli_logit(alpha) == bernoulli(inv_logit(alpha))
```

#### Proposed

Keep as is.


#### Discussion

* Original plan from Aki was to do this:

```
poisson<link=log>(alpha)
bernoulli<link=logit>(alpha)
```

* But that's verbose and then have to deal with multiple argument type names

* User's may overgeneralize from the general syntax to assume arguments that don't exist
    * so just use `bernoulli_logit` instead of `bernoulli<link=logit>`


## User-Defined Functions

#### Current

Use current versions of function names, with `_log` allowing sampling statement use and "_lp" allowing access to log density.

#### Proposed

1. Use proposed versions of suffixes, with `_lpdf` and `_cdf` and `_ccdf` and `_diff_cdf` (when we get to it) supporting vertical bar notation.

1.  Use `_target` suffix instead of `_lp` (deprecate `_lp`) to allow access to `target`

1. Deprecate (not eliminate for now) use of `_log` and `_lp` suffixes in user-defined functions.

#### Discussion

* I think everyone's OK with this.