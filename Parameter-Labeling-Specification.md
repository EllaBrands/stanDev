Not all parameters are created equal. 

**Goal:** Provide labeling functionality for parameters. We can then treat parameters with special labels differently in our inference algorithms.

## Example Usage for ADVI

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

## Example Usage for MML

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
We want to estimate the hyperparameters `phi`, integrated over the local parameters `alpha`.

**Related notes:** [Max Marginal Optimization (lmer) Design](https://github.com/stan-dev/stan/wiki/Max-Marginal-Optimization-(lmer)-Design), [MLE and MML Design](https://github.com/stan-dev/stan/wiki/MLE-and-MML-Design)

## Proposal

### Stan file labeling

We would like to specify the labels at runtime, based on arguments depending on each inference algorithm. We do this similar to how we specify inits.

For ADVI, consider the following:

```bash
./foo advi meanfield=“w” fullrank=“alpha, sigma”
```

All parameters labeled with `fullrank` will get assigned to a full-rank variational approximation. In this case, that's a `D+1` dimensional multivariate Gaussian with `bigO( (D+1)^2 )` variational parameters.

All parameters labeled with `meanfield` (or left unlabeled) will get assigned to a mean-field variational approximation. In this case, that's a `D` dimensional diagonal Gaussian with `bigO( D )` variational parameters.

A similar interface applies to maximum marginal likelihood.
```bash
./foo mml hyperparameters=“phi" local="alpha"
```

### Model implementation

See however init does it.

### (archives)
Consider the following labeling scheme.

```C++
parameters {
  vector<lower=0,#fullrank>[D] alpha; // hierarchical latent variables
  real<lower=0,#fullrank> sigma;      // standard deviation
  vector<#meanfield>[D] w;            // weights (coefficients) vector
}
```
We would want a function similar to `get_param_names` that writes the label names out.

```C++
void get_label_names(std::vector<std::string>& labels) const;
```

On the ADVI side, we would build a hybrid variational family
```C++
src/stan/variational/families/normal_hybrid.hpp
```
that would use these labels in a meaningful way.

**NOTE** We don't need to touch how `cont_params_` work. Nor do we need to play with the gradients at all. We would still only need `model_.template log_prob<false, true>` and `stan::model::gradient` for ADVI.