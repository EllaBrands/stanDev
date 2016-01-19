Each probability distribution, such as Bernoulli or normal, has functions for its probability density function (PDF), cumulative distribution function (CDF), complementary cumulative distribution function (CCDF), inverse CDFs (ICDF), and pseudo random number generator (RNG)

## Function Suffixes

To distinguish which function for a distribution is being used and which scale.

#### Current Suffixes

scale      | PDF   | CDF     | CCDF     | inverse CDF | PRNG 
-----------|-------|---------|----------|-------------|------
*linear*   | n/a   | cdf     | ccdf     |    n/a      | rng   
*log*      | log   | cdf_log | ccdf_log |    n/a      | n/a

#### Proposed Suffixes

scale    | PDF   | CDF  | CCDF  | diff of CDFs | inv CDF  | PRNG
---------|-------|------|-------|--------------|----------|-----
*linear* | pdf   | cdf  | ccdf  | diff_cdf     | inv_cdf  | rng
*log*    | lpdf  | lcdf | lccdf | ldiff_cdf    | n/a      | n/a     

e.g., normal_pdf, normal_lpdf, normal_cdf, normal_lcdf, normal_ccdf, normal_lccdf

Deprecate (not eliminate) existing functions.

##### Discussion

* We should think of PMFs as PDFs.

* For inverse CDF, is the argument in (0, 1), or is it log odds or log prob?

* Andrew suggests we should only supply the log functions where relevant and drop the "l" suffix, so that's

scale   | PDF   | CDF  | CCDF  | diff of CDFs
--------|-------|------|-------|-------------
*log*   |  pdf  |  cdf |  ccdf | diff_cdf

and then

scale    | RNG | inv CDF
---------|-----|--------
*linear* | rng | icdf


## Distribution Name Defaults to Log Density

#### Proposal

Writing `normal(y | mu, sigma)` would produce the same value as `normal_lpdf(y | mu, sigma)`

#### Discussion 

* Not good to have two ways to write the same thing

* Removing `normal_lpdf` then removes the symmetry, so it's

```
normal(y | mu, sigma)
normal_cdf(y | mu, sigma)
normal_ccdf(y | mu, sigma)
normal_diff_cdf(y_l, y_h | mu, sigma)
normal_inv_cdf(p, mu, sigma)
```

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
cauchy_lpdf<norm=true>(y | mu, tau);   // normalized
cauchy_lpdf<norm=false>(y | mu, tau);  // unnormalized
cauchy_lpdf(y | mu, tau);              // normalized
```

#### Discussion

* The default is *normalized*
    * this is going to lead to a lot of inefficiency with users
    * it's going to lead to confusion for users about where it's needed
    * what users will expect of a function

* It would be great to only compute normalizing constants once on outside and use all normalized inside, but
    * won't work if there is branching on parameters
    * need the values on the inside for mixture models

* Aki suggests following Andrew's *BDA* notation with `q()` being unnormalized form of `p()`, so
    * instead of `beta_lpdf<norm=true>(y | a, b)`, write `beta_lqdf(y | a, b)`
    * could be confusing given `q` usage for quantiles in R
    

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

1.  `ld << normal_lpdf(y | mu, sigma);`

The `ld` is for "log density".

#### Discussion

* Streaming operator `<<` unfamiliar outside of C++

* Alternative: `+=` 
    * familiar from C/C++/Java/Python (but not in R or BUGS or JAGS).
    * for consistency, allow `+=` for other variables
    * disallow other operations on `ld`
    * for consistency, replace `<-` with `=`, too

* `ld` is abstract here, at least until there are more
    * could allow just `<<` without any `ld` on left-hand side

* reserve `ld` (not strictly necessary)

* later, we might allow comma-separated values on right-hand side as in Eigen matrix assignment, e.g.,
```
ld << normal_lpdf(mu | 0, 1), 
      normal_lpdf(beta | mu, sigma), 
      cauchy_lpdf(sigma | 0, 2.5);
```
But it's arguably confusing, and the following alternative will be available.
```
ld << normal_lpdf(mu | 0, 1);
ld << normal_lpdf(beta | mu, sigma);
ld << cauchy_lpdf(sigma | 0, 2.5);
```

* Could avoid streaming altogether and use a random prefix like:
```
@normal_lpdf(mu | 0, 1);
@normal_lpdf(beta | mu, sigma);
@cauchy_lpdf(sigma | 0, 2.5);
```

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

## User-Defined Functions

#### Current

Use current versions of function names, with `_log` allowing sampling statement use and "_lp" allowing access to log density.

#### Proposed

Use proposed versions of suffixes, with `_pdf` supporting vertical bar notation.

Use "_ld" instead of "_lp" in user-defined functions with access to increment-log-density streaming.

Deprecate (not eliminate) use of `_log` in sampling statements.