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

## Proposal

### Stan file labeling

Consider the following labeling scheme.

```C++
parameters {
  vector<lower=0,#fullrank>[D] alpha; // hierarchical latent variables
  real<lower=0,#fullrank> sigma;      // standard deviation
  vector<#meanfield>[D] w;            // weights (coefficients) vector
}
```

All parameters labeled with `#fullrank` will get assigned to a full-rank variational approximation. In this case, that's a `D+1` dimensional multivariate Gaussian with `bigO( (D+1)^2 )` variational parameters.

All parameters labeled with `#meanfield` (or left unlabeled) will get assigned to a mean-field variational approximation. In this case, that's a `D` dimensional diagonal Gaussian with `bigO( D )` variational parameters.

### Model implementation

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

