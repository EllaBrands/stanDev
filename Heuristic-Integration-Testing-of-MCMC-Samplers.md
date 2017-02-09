In order to evaluate the current version of the Stan library, or proposed changes thereto, for valid statistical estimation one has to consider not just the MCMC estimators but also their expected variation.  Even in optimal circumstances, the one guarantee that an MCMC algorithm makes is that (f_hat - E[f]) / MCMC-SE ~ N(0, 1), where f_hat is the MCMC estimator of the expectation E[f] and MCMC-SE is the MCMC standard error.  Consequently any heuristic evolution should follow a general pattern:

1. Select and run a Stan program to evaluate using any interface.  Multiple chains is highly encouraged.

2. Ensure that all diagnostics are negative.  This means at the very minimum that all split Rhats are below 1.1, there are no divergences, and the E-BFMI is below 0.3.

3. For relevant functions, relevant here being up to the discretion of the evaluator, look at Delta = (f_hat - E[f]) / MCMC-SE and compare to the distribution N(0, 1).

4. If any values are suspiciously large then rerun with 10 times more iterations.  This reduces the MCMC-SE by a factor of sqrt(10) and hence the expected fluctuations in Delta. Any true bias, however, will persist and hence grow in relative importance.

An example model to evaluate is included below.  Note that both some variances and a correlation are computed in the generated quantities block so that their MCMC-SE will also be computed and they can be used in evaluations.  This has proven useful in the past when facing bugs that induce bias not in means but in variances.
```
transformed data {
  vector[2] mu;
  real sigma1;
  real sigma2;
  real rho;

  matrix[2,2] Sigma;

  mu[1] = 0.0;
  mu[2] = 3.0;

  rho = 0.5;
  sigma1 = 1.0;
  sigma2 = 2.0;

  Sigma[1][1] = sigma1 * sigma1;
  Sigma[1][2] = rho * sigma1 * sigma2;
  Sigma[2][1] = rho * sigma1 * sigma2;
  Sigma[2][2] = sigma2 * sigma2;
}

parameters {
  vector[2] z;
}

model {
  z ~ multi_normal(mu, Sigma);
}

generated quantities {
  // The means of these quantities will give the difference between 
  // estimated marginal variances and correlation and their true values.
  // If everything is going correctly then these values should be with
  // the respective MCMC-SE of zero.
  real delta_var1;
  real delta_var2;
  real delta_corr;

  delta_var1 = square(z[1] - mu[1]) - sigma1 * sigma1;
  delta_var2 = square(z[2] - mu[2]) - sigma2 * sigma2;
  delta_corr = (z[1] - mu[1]) * (z[2] - mu[2]) / (sigma1 * sigma2) - rho;
}
```