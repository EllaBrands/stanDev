Not all parameters are created equal. 

**Goal:** Provide labeling functionality for parameters. We can then treat parameters with special labels differently in our inference algorithms.

_Disclaimer_: This is a very rough work in progress. It does not detail a concrete (or good) solution. Rather, it itemizes the general extensions of current algorithms that cannot currently be implemented.

## Motivation for MML

Consider the following hierarchical model.
```C++
data {
  int<lower=1> N;
  int<lower=1> J;
  int<lower=1> K;
  real y[N];
  int<lower=1, upper=J> group[N];
}
parameters {
  vector[K] phi;              // mu, log_sigma_y, log_sigma_alpha
  vector[J] alpha;
}
transformed parameters {
  real mu;
  real<lower=0> sigma_y;
  real<lower=0> sigma_alpha;
  mu <- phi[1];
  sigma_y <- exp(phi[2]);
  sigma_alpha <- exp(phi[3]);
}
model {
  real y_pred[N];
  for (n in 1:N){
    y_pred[n] <- mu + alpha[group[n]];
  }
  increment_log_prob(normal_log(y, y_pred, sigma_y));
  increment_log_prob(normal_log(alpha, 0, sigma_alpha));
  increment_log_prob(log(sigma_y));
  increment_log_prob(log(sigma_alpha));
  print(lp__);
}
```
We want to estimate the hyperparameters `phi`, integrated over the local parameters `alpha`. To do so, the user needs somehow to signal to the program which are parameters to optimize over and which are parameters to integrate over.

**Related notes:** [Max Marginal Optimization (lmer) Design](https://github.com/stan-dev/stan/wiki/Max-Marginal-Optimization-(lmer)-Design), [MLE and MML Design](https://github.com/stan-dev/stan/wiki/MLE-and-MML-Design)

## Motivation for Discrete Parameters

We have no way to distinguish between discrete and continuous parameters. We need this before we can start implementing a inference algorithm for models with discrete parameters.

## Motivation for ADVI

Assume `K` parameters in a model.

Mean-field uses `2K` variational parameters for the fully factorized approximation.

Full-rank uses `K(K-1)/2` variational parameters for the multivariate approximation.

Consider the following hierarchical model. 

```C++
data {
  int<lower=0> N;   // number of data items
  int<lower=0> D;   // dimension of input features
  matrix[N,D]  x;   // input matrix
  vector[N]    y;   // output vector
}

parameters {
  vector<lower=0>[D] alpha; // hierarchical latent variables
  real<lower=0> sigma;      // standard deviation
  vector[D] w;              // weights (coefficients) vector
}

transformed parameters {
  vector[D] one_over_sqrt_alpha;
  for (i in 1:D) {
    one_over_sqrt_alpha[i] <- 1 / sqrt(alpha[i]);
  }
}

model {
  alpha ~ gamma(0.5, 0.5);
  sigma ~ gamma(0.5, 0.5);
  w ~ normal(0, sigma * one_over_sqrt_alpha);
  y ~ normal(x * w, sigma);
}
```

It makes sense to use a full-rank approximation for the hierarchical parameters `alpha`, but use a mean-field approximation for the low-level parameters `w`. 

## Proposal

### Stan file labeling

We would like to specify the labels at runtime, based on arguments depending on each inference algorithm. We do this similar to how we specify inits.

A similar interface applies to maximum marginal likelihood.
```bash
./foo mml hyperparameters=“phi" local="alpha"
```

For ADVI, consider the following:

```bash
./foo advi meanfield=“w” fullrank=“alpha, sigma”
```

All parameters labeled with `fullrank` will get assigned to a full-rank variational approximation. All parameters labeled with `meanfield` (or left unlabeled) will get assigned to a mean-field variational approximation.

### Implementation

This is done similarly to how the `init` argument for inference algorithms is done.