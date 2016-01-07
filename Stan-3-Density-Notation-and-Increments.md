Each probability distribution, such as Bernoulli or normal, has functions for its probability density function (PDF), cumulative distribution function (CDF), complementary cumulative distribution function (CCDF), inverse CDFs (ICDF), and pseudo random number generator (RNG)

### Function Suffixes

To distinguish which function for a distribution is being used and which scale.

#### Current Suffixes

Scale | PDF   | CDF     | CCDF     | inverse CDF | PRNG 
------|-------|---------|----------|-------------|------
Linear| n/a   | cdf     | ccdf     |    n/a      | rng   
Log   | log   | cdf_log | ccdf_log |    n/a      | n/a

#### Proposed Suffixes

Scale | PDF   | CDF  | CCDF  | inv CDF    | PRNG
------|-------|------|-------|------------|-------
Linear| pdf   | cdf  | ccdf  | icdf       | rng
Log   | lpdf  | lcdf | lccdf | n/a        | n/a

e.g., normal_pdf, normal_lpdf, normal_cdf, normal_lcdf, normal_ccdf, normal_lccdf

Deprecate (not eliminate) existing functions.

Andrew suggests we only supply the log functions (RNG is a bit different here) and not use the "l" suffix., so that'd be: 

Scale | PDF   | CDF  | CCDF  |
------|-------|------|-------|
Log   |  pdf  |  cdf |  ccdf |

and then

Scale  | RNG | inv CDF
-------|-----|--------
linear | rng | icdf

##### Discussion

We should think of PMFs as PDFs.

We eventually want differences of cdfs (dcdf).  


### Vertical Bar Notation

To distinguish conditional densities from joint densities in the notation.

#### Current

`normal_log(y, mu, sigma)`

#### Proposed

`normal_lpdf(y | mu, sigma)`

##### Discussion

May be confusing because it's unconventional in programming languages.


### Normalization Control

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
cauchy_lpdf(y | mu, tau);              // unnormalized
```

### Scalar/vector Output Control

#### Current 

If y is vector (or array) sampling statement and function call compute sum of log densities
```
y ~ normal(mu, sigma);
sum_log_lik <- normal_log(y, mu, sigma)
```

In generative quantities it is common to have:
```
for (n in 1:N)
 log_lik[n] <- normal_log(y[n], mu, sigma)
```

#### Proposed

PDF has special arguments for vector output, defaulting to `false`:

```
log_lik <- normal_lpdf<norm=true,vector=true>(y | mu, tau);   // normalized and vector output
```

or PDF has special arguments for summing, defaulting to `true`:

```
log_lik <- normal_lpdf<norm=true,sum=false>(y | mu, tau);   // normalized and not summing
```


### Increment Log Density Statement

Make it clear that we're incrementing an underlying accumulator rather than forward sampling or defining a directed graph.

#### Current

1.  `increment_log_prob(normal_log(y, mu, sigma));`

1.  `y ~ normal(mu, sigma);`

#### Proposed

Replace both of these with a single version:

1.  `ld << normal_lpdf(y | mu, sigma);`

The `ld` is for "log density".

##### Discussion

* Streaming operator `<<` unfamiliar outside of C++
    * `+=` is familiar from C/C++/Java/Python (but not in R or BUGS or JAGS).

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


### User-Defined Functions

#### Current

Use current versions of function names, with `_log` allowing sampling statement use and "_lp" allowing access to log density.

#### Proposed

Use proposed versions of suffixes, with `_pdf` supporting vertical bar notation.

Use "_ld" instead of "_lp" in user-defined functions with access to increment-log-density streaming.

Deprecate (not eliminate) use of `_log` in sampling statements.