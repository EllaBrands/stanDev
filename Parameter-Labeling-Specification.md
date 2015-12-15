# Parameter Labeling Specification

Not all parameters are created equal. 

**Goal:** Provide labeling functionality for parameters. We can then treat parameters with special labels differently in our inference algorithms.

## Example Usage in ADVI

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