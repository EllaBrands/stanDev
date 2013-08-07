***

***

## Single-Level Models

### Linear Models

#### No Predictors

   * [lightspeed](https://github.com/stan-dev/stan/blob/feature/ARM/src/models/ARM/Ch.8/lightspeed.stan): linear model with no predictors
```
lm (y ~ 1)
```

#### One Predictor

   * [earnings](https://github.com/stan-dev/stan/blob/feature/ARM/src/models/ARM/Ch.4/earn_height.stan): linear model with one predictor  
```
lm (earnings ~ height)
```

   * [earnings](https://github.com/stan-dev/stan/blob/feature/ARM/src/models/ARM/Ch.4/logearn_height.stan): linear model with one predictor and natural log transformation  
```
lm (log_earnings ~ height)
```

   * [earnings](https://github.com/stan-dev/stan/blob/feature/ARM/src/models/ARM/Ch.4/log10earn_height.stan): linear model with one predictor and log10 transformation  
```
lm (log10_earnings ~ height)
```

   * [electric](https://github.com/stan-dev/stan/blob/feature/ARM/src/models/ARM/Ch.9/electric_one_pred.stan): linear model with one predictor
```
lm (post_test ~ treatment)
```

   * [grades](https://github.com/stan-dev/stan/blob/feature/ARM/src/models/ARM/Ch.8/grades.stan): linear model with one predictor
```
lm (final ~ midterm)
```

   * [kid_iq](https://github.com/stan-dev/stan/blob/feature/ARM/src/models/ARM/Ch.3/kidscore_momhs.stan): linear model with one predictor  
```
lm (kid_score ~ mom_hs)
```

   * [kid_iq](https://github.com/stan-dev/stan/blob/feature/ARM/src/models/ARM/Ch.3/kidscore_momiq.stan): linear model with one predictor  
```
lm (kid_score ~ mom_iq)
```

#### Multiple Predictors with No Interaction

   * [congress](https://github.com/stan-dev/stan/blob/feature/ARM/src/models/ARM/Ch.7/congress.stan): linear model with two predictors  
```
lm (vote_88 ~ vote_86 + incumbency_88)
```

   * [earnings](https://github.com/stan-dev/stan/blob/feature/ARM/src/models/ARM/Ch.4/logearn_height_male.stan): linear model with two predictors and natural log transformation  
```
lm (log_earnings ~ height + male)
```

   * [earnings](https://github.com/stan-dev/stan/blob/feature/ARM/src/models/ARM/Ch.4/logearn_logheight.stan): linear model with two predictors and log log transformation  
```
lm (log_earnings ~ log_height + male)
```

   * [earnings](https://github.com/stan-dev/stan/blob/feature/ARM/src/models/ARM/Ch.6/earnings2.stan): linear model with two predictors and log transformation 
```
lm (log_earn ~ height + male, subset=earn>0)
```

   * [earnings2](https://github.com/stan-dev/stan/blob/feature/ARM/src/models/ARM/Ch.25/earnings2.stan): mlinear model with eleven predictors
```
lm (earnings ~ interest + male + over65 + white + immig + educ_r + workmos + workhrs_top + any_ssi 
               + any_welfare + any_charity)
```

   * [earnings](https://github.com/stan-dev/stan/blob/feature/ARM/src/models/ARM/Ch.25/earnings.stan): linear model with ten predictors
```
lm (earnings ~ male + over65 + white + immig + educ_r + workmos + workhrs_top + any_ssi + any_welfare 
               + any_charity)
```

   * [earnings_pt2](https://github.com/stan-dev/stan/blob/feature/ARM/src/models/ARM/Ch.25/earnings.stan): linear model with eight predictors
```
lm (earnings ~ male + over65 + white + immig + educ_r + any_ssi + any_welfare + any_charity)
```

   * [electric](https://github.com/stan-dev/stan/blob/feature/ARM/src/models/ARM/Ch.9/electric_multi_preds.stan): linear model with two predictors  
```
lm (post_test ~ pre_test + treatment)
```

   * [ideo](https://github.com/stan-dev/stan/blob/feature/ARM/src/models/ARM/Ch.10/ideo_two_pred.stan): linear model with two predictors  
```
lm (score1 ~ party + x, subset=overlap)
```

   * [ideo](https://github.com/stan-dev/stan/blob/feature/ARM/src/models/ARM/Ch.10/ideo_two_pred.stan): linear model with two predictors  
```
lm (score1 ~ party + x, subset=incs)
```

   * [ideo](https://github.com/stan-dev/stan/blob/feature/ARM/src/models/ARM/Ch.10/ideo_interactions.stan): linear model with two predictors and reparamaterization  
```
lm (score1 ~ party + I(z*(party==0)) + I(z*(party==1)), subset=incs)
```

   * [kid_iq](https://github.com/stan-dev/stan/blob/feature/ARM/src/models/ARM/Ch.3/kidiq_multi_preds.stan): linear model with two predictors  
```
lm (kid_score ~ mom_hs + mom_iq)
```

   * [kid_iq](https://github.com/stan-dev/stan/blob/feature/ARM/src/models/ARM/Ch.3/kidiq_validation.stan): linear model with two predictors  
```
lm (ppvt ~ hs + afqt)
```

   * [kid_iq](https://github.com/stan-dev/stan/blob/feature/ARM/src/models/ARM/Ch.4/kidscore_momwork.stan): linear model with one factor   
```
lm (kid_score ~ as.factor(mom_work))
```

   * [mesquite](https://github.com/stan-dev/stan/blob/feature/ARM/src/models/ARM/Ch.4/mesquite.stan): linear model with six predictors  
```
lm (weight~ diam1 + diam2 + canopy_height + total_height + density + group)
```

   * [mesquite](https://github.com/stan-dev/stan/blob/feature/ARM/src/models/ARM/Ch.4/mesquite_log.stan): linear model with six predictors and log transformation  
```
lm (log_weight~ log_diam1 + log_diam2 + log_canopy_height + log_total_height + log_density + group)
```

   * [nes](https://github.com/stan-dev/stan/blob/feature/ARM/src/models/ARM/Ch.4/nes.stan): linear model with eight predictors    
```
lm (partyid7 ~ real_ideo + race_adj + age30_44 + age45_64 + age65up + educ1 + gender + income)
```

#### Multiple Predictors with Interaction

   * [earnings](https://github.com/stan-dev/stan/blob/feature/ARM/src/models/ARM/Ch.4/logearn_interaction.stan): linear model with two predictors and interaction and natural log transformation  
```
lm (log_earnings ~ height + male + height:male)
```

   * [earnings](https://github.com/stan-dev/stan/blob/feature/ARM/src/models/ARM/Ch.4/logearn_interaction_z.stan): linear model with two predictors and interaction and natural log transformation centered using z-score  
```
lm (log_earnings ~ z_height + male + z_height:male)
```

   * [electric](https://github.com/stan-dev/stan/blob/feature/ARM/src/models/ARM/Ch.9/electric_interactions.stan): linear model with two predictors and interaction  
```
lm (post_test ~ pre_test + treatment + pre_test:treatment)
```

   * [ideo](https://github.com/stan-dev/stan/blob/feature/ARM/src/models/ARM/Ch.10/ideo_reparam.stan): linear model with two predictors and interaction  
```
lm (score1 ~ party + x + party:x, subset=incs)
```

   * [kid_iq](https://github.com/stan-dev/stan/blob/feature/ARM/src/models/ARM/Ch.3/kidiq_interaction.stan): linear model with two predictors and interaction  
```
lm (kid_score ~ mom_hs + mom_iq + mom_hs:mom_iq)
```

   * [kid_iq](https://github.com/stan-dev/stan/blob/feature/ARM/src/models/ARM/Ch.4/kidiq_interaction_c.stan): linear model with two predictors and interaction centered using mean  
```
lm (kid_score ~ c_mom_hs + c_mom_iq + c_mom_hs:c_mom_iq)
```

   * [kid_iq](https://github.com/stan-dev/stan/blob/feature/ARM/src/models/ARM/Ch.4/kidiq_interaction_c2.stan): linear model with two predictors and interaction centered using conventional points  
```
lm (kid_score ~ c2_mom_hs + c2_mom_iq + c2_mom_hs:c2_mom_iq)
```

   * [kid_iq](https://github.com/stan-dev/stan/blob/feature/ARM/src/models/ARM/Ch.4/kidiq_interaction_z.stan): linear model with two predictors and interaction centered using z-score  
```
lm (kid_score ~ z_mom_hs + z_mom_iq + z_mom_hs:z_mom_iq)
```

   * [mesquite](https://github.com/stan-dev/stan/blob/feature/ARM/src/models/ARM/Ch.4/mesquite_volume.stan): linear model with one transformed predictor and log transformation  
```
lm (log_weight ~ log_canopy_volume)
```

   * [mesquite](https://github.com/stan-dev/stan/blob/feature/ARM/src/models/ARM/Ch.4/mesquite_vas.stan): linear model with three predictors and three transformed predictors and log transformation           
```
lm (log_weight ~ log_canopy_volume + log_canopy_area + log_canopy_shape + log_total_height + log_density + group)
```

   * [mesquite](https://github.com/stan-dev/stan/blob/feature/ARM/src/models/ARM/Ch.4/mesquite_va.stan): linear model with one predictor and two transformed predictors and log transformation   
```
lm (log_weight ~ log_canopy_volume + log_canopy_area + group)
```

   * [mesquite](https://github.com/stan-dev/stan/blob/feature/ARM/src/models/ARM/Ch.4/mesquite_vash.stan): linear model with two predictors and three transformed predictors and log transformation
```
lm (log_weight ~ log_canopy_volume + log_canopy_area + log_canopy_shape + log_total_height + group)
```

### Logit Regression Models

#### One Predictor

#### Multiple Predictors with No Interaction

   * [earnings](https://github.com/stan-dev/stan/blob/feature/ARM/src/models/ARM/Ch.6/earnings1.stan): generalized linear model with logit link function and two predictors  
```
glm (earn_pos ~ height + male, family=binomial(link="logit"))
```

   * [earnings_pt1](https://github.com/stan-dev/stan/blob/feature/ARM/src/models/ARM/Ch.25/earnings.stan): logistic regression model with eight predictors
```
glm (earnings ~ male + over65 + white + immig + educ_r + any_ssi + any_welfare + any_charity,
     family=binomial(link="logit"))
```

#### Multiple Predictors with Interaction








***
***
***

#### One Predictor

#### Multiple Predictors with No Interaction

#### Multiple Predictors with Interaction



 