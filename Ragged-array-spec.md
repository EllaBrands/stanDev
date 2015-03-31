## Ragged array spec

### Use cases

#### Use case 1: Varying intercept model

Current Stan model for a varying intercept model:

```
data {
  int<lower=1> N; // number of observations
  int<lower=1> K; // number of random effects
  // for each category with a random effect, number 
  // of levels within a category
  // e.g. the state category would have 51 levels 
  int<lower=1> levels_per_category[K];
  // per category, observation-level assignment to a level
  // for category k
  int<lower=1> unit_level_id[K,N];
}
parameters {
  vector<lower=0>[K] std_re;
  real<lower=0> obs_std;
  real alpha;
  real<lower=0> std_std;
  vector[levels_per_category[1]] re_group_1;
  vector[levels_per_category[2]] re_group_2;
  vector[levels_per_category[3]] re_group_2;
  ...
  ...
  vector[levels_per_category[K]] re_group_K;
}
model {
  vector[N] unit_level_mean;
  for(n in 1:N)
    unit_level_mean[n] <- alpha + re_group_1[unit_level_id[1,n]] + ... 
                          + re_group_K[unit_level_id[K,n]];
  y ~ normal(unit_level_mean, obs_std);
  re_group_1 ~ normal(0, std_re[1]);
  re_group_2 ~ normal(0, std_re[2]);
  ...
  re_group_K ~ normal(0, std_re[K]);
  std_re ~ normal(0,std_std);
  obs_std ~ normal(0,1);
  std_std ~ normal(0,1);
}
```

Ideally with ragged arrays I would like to be able to write the parameter block and model block like so:

```
parameters {
  vector<lower=0>[K] std_re;
  real<lower=0> obs_std;
  real alpha;
  real<lower=0> std_std;
  ragged_matrix[levels_per_category,rep_array(1,K)] re_group[K];
}
model {
  vector[N] unit_level_mean;
  vector[K] accum;
  for(n in 1:N){
    for(k in 1:K) 
      accum[k] <- re_group[k,group_id[k,n]]
    unit_level_mean[n] <- alpha + sum(accum);
  }  
  y ~ normal(unit_level_mean, obs_std);
  for(k in 1:K)
    re_group[k] ~ normal(0,std_re[k]);
  std_re ~ normal(0,std_std);
  obs_std ~ normal(0,1);
  std_std ~ normal(0,1);
}
```

#### Use case 2: Random effects models with covariates

w/o ragged arrays:

```
data {
  int<lower=1> N; // number of observations
  int<lower=1> K; // number of random effects
  // for each category with a random effect, number 
  // of levels within a category
  // e.g. the state category would have 51 levels 
  int<lower=1> levels_per_category[K];
  int<lower=1> unit_level_id[K,N];
  int<lower=1> X_col_by_category[K];
  matrix[levels_per_category[1],X_col_by_category[1]] X_category_1;
  ...
  matrix[levels_per_category[K],X_col_by_category[K]] X_category_K;
}
parameters {
  vector<lower=0>[K] std_re;
  real<lower=0> obs_std;
  real alpha;
  real<lower=0> std_std;
  vector[levels_per_category[1]] re_group_1;
  vector[levels_per_category[2]] re_group_2;
  vector[levels_per_category[3]] re_group_2;
  ...
  ...
  vector[levels_per_category[K]] re_group_K;
  vector[X_col_by_category[1]] beta_1;
  ...
  vector[X_col_by_cateogry[K]] beta_K;
}
model {
  vector[N] unit_level_mean;
  for(n in 1:N)
    unit_level_mean[n] <- re_group_1[unit_level_id[1,n]] + ... 
                          + re_group_K[unit_level_id[K,n]];
  y ~ normal(unit_level_mean, obs_std);
  re_group_1 ~ normal(X_category_1 * beta_1, std_re[1]);
  re_group_2 ~ normal(X_category_2 * beta_2, std_re[2]);
  ... 
  re_group_K ~ normal(X_category_K * beta_K, std_re[K]);
  std_re ~ normal(0,std_std);
  obs_std ~ normal(0,1);
  std_std ~ normal(0,1);
  beta_1 ~ normal(0,1);
  ...
  beta_K ~ normal(0,1);
}
```
w ragged arrays:
```
data {
  int<lower=1> N; // number of observations
  int<lower=1> K; // number of random effects
  // for each category with a random effect, number 
  // of levels within a category
  // e.g. the state category would have 51 levels 
  int<lower=1> levels_per_category[K];
  int<lower=1> unit_level_id[K,N];
  // each category has an X matrix dimension 
  // row_k (levels_per_category[k]) 
  // and col_k, X_col_by_category[k] = col_k
  int<lower=1> X_col_by_category[K];
  ragged_matrix[levels_per_category,X_col_by_category] X[K];
}
parameters {
  vector<lower=0>[K] std_re;
  real<lower=0> obs_std;
  real alpha;
  real<lower=0> std_std;
  ragged_matrix[levels_per_category,rep_array(1,K)] re_group[K];
  ragged_matrix[X_col_by_category,rep_array(1,K)] beta;
}
model {
  vector[N] unit_level_mean;
  vector[K] accum;
  for(n in 1:N){
    for(k in 1:K) 
      accum[k] <- re_group[k,group_id[k,n]]
    unit_level_mean[n] <- alpha + sum(accum);
  }  
  y ~ normal(unit_level_mean, obs_std);
  for(k in 1:K){
    re_group[k] ~ normal(X[k] * beta[k],std_re[k]);
    beta[k] ~ normal(0,1);
  }
  std_re ~ normal(0,std_std);
  obs_std ~ normal(0,1);
  std_std ~ normal(0,1);
}
```