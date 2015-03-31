### Ragged array spec

#### Use case 1: Varying intercept model

Current Stan model for a varying intercept model:

```
data {
  int<lower=1> N; // number of observations
  int<lower=1> K; // number of random effects
  // for each category with a random effect, number 
  // of levels within a category
  // e.g. the state category would have 51 levels 
  int<lower=1> num_levels_per_category[K];
  int<lower=1> level_id_per_category[K,N];
}
parameters {
  vector<lower=0>[K] std_re;
  real<lower=0> obs_std;
  real alpha;
  real<lower=0> std_std;
  vector[num_groups[1]] re_group_1;
  vector[num_groups[2]] re_group_2;
  vector[num_groups[3]] re_group_2;
  ...
  ...
  vector[num_groups[K]] re_group_K;
}
model {
  vector[N] unit_level_mean;
  for(n in 1:N)
    unit_level_mean[n] <- alpha + re_group_1[level_id_per_category[1,n]] + ... 
                          + re_group_K[level_id_per_category[K,n]];
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
  ragged_matrix[num_levels_per_category,rep_array(1,K)] re_group[K];
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