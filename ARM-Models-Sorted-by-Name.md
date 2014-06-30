* [8 Schools](https://github.com/stan-dev/stan/wiki/ARM-Models-Sorted-by-Name#8-schools)
* [Congress](https://github.com/stan-dev/stan/wiki/ARM-Models-Sorted-by-Name#congress)
* [Dogs](https://github.com/stan-dev/stan/wiki/ARM-Models-Sorted-by-Name#dogs)
* [Earnings](https://github.com/stan-dev/stan/wiki/ARM-Models-Sorted-by-Name#earnings)
* [Election88](https://github.com/stan-dev/stan/wiki/ARM-Models-Sorted-by-Name#election88)
* [Electric](https://github.com/stan-dev/stan/wiki/ARM-Models-Sorted-by-Name#electric)
* [Grades](https://github.com/stan-dev/stan/wiki/ARM-Models-Sorted-by-Name#grades)
* [HIV](https://github.com/stan-dev/stan/wiki/ARM-Models-Sorted-by-Name#hiv)
* [Ideo](https://github.com/stan-dev/stan/wiki/ARM-Models-Sorted-by-Name#ideo)
* [Item Response](https://github.com/stan-dev/stan/wiki/ARM-Models-Sorted-by-Name#item-response)
* [Kid IQ](https://github.com/stan-dev/stan/wiki/ARM-Models-Sorted-by-Name#kid-iq)
* [Lightspeed](https://github.com/stan-dev/stan/wiki/ARM-Models-Sorted-by-Name#lightspeed)
* [Mesquite](https://github.com/stan-dev/stan/wiki/ARM-Models-Sorted-by-Name#mesquite)
* [NES](https://github.com/stan-dev/stan/wiki/ARM-Models-Sorted-by-Name#nes)
* [Pilots](https://github.com/stan-dev/stan/wiki/ARM-Models-Sorted-by-Name#pilots)
* [Radon](https://github.com/stan-dev/stan/wiki/ARM-Models-Sorted-by-Name#radon)
* [Roaches](https://github.com/stan-dev/stan/wiki/ARM-Models-Sorted-by-Name#roaches)
* [Sesame](https://github.com/stan-dev/stan/wiki/ARM-Models-Sorted-by-Name#sesame)
* [Statistical Calculations](https://github.com/stan-dev/stan/wiki/ARM-Models-Sorted-by-Name#statistical-calculations)
* [Unemployment](https://github.com/stan-dev/stan/wiki/ARM-Models-Sorted-by-Name#unemployment)
* [Wells](https://github.com/stan-dev/stan/wiki/ARM-Models-Sorted-by-Name#wells)

***

***

#### 8 Schools

   * [8_schools](https://github.com/stan-dev/stan/blob/feature/ARM/src/models/ARM/Ch.19/schools.stan): multi-level linear model with redundant parameterization

***

#### Congress

   * [congress](https://github.com/stan-dev/stan/blob/feature/ARM/src/models/ARM/Ch.7/congress.stan): linear model with two predictors  
```
lm (vote_88 ~ vote_86 + incumbency_88)
```
 
*** 

#### Dogs

   * [dogs](https://github.com/stan-dev/stan/blob/feature/ARM/src/models/ARM/Ch.24/dogs.stan): multi-level logit regression model

   * [dogs_log](https://github.com/stan-dev/stan/blob/feature/ARM/src/models/ARM/Ch.24/dogs_log.stan): multi-level model using binomial distribution

   * [dogs_check](https://github.com/stan-dev/stan/blob/feature/ARM/src/models/ARM/Ch.24/dogs_check.stan): multi-level model using binomial distribution

***

#### Earnings

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

   * [earnings](https://github.com/stan-dev/stan/blob/feature/ARM/src/models/ARM/Ch.4/logearn_height_male.stan): linear model with two predictors and natural log transformation  
```
lm (log_earnings ~ height + male)
```

   * [earnings](https://github.com/stan-dev/stan/blob/feature/ARM/src/models/ARM/Ch.4/logearn_interaction.stan): linear model with two predictors and interaction and natural log transformation  
```
lm (log_earnings ~ height + male + height:male)
```

   * [earnings](https://github.com/stan-dev/stan/blob/feature/ARM/src/models/ARM/Ch.4/logearn_interaction_z.stan): linear model with two predictors and interaction and natural log transformation centered using z-score  
```
lm (log_earnings ~ z_height + male + z_height:male)
```

   * [earnings](https://github.com/stan-dev/stan/blob/feature/ARM/src/models/ARM/Ch.4/logearn_logheight.stan): linear model with two predictors and log log transformation  
```
lm (log_earnings ~ log_height + male)
```

   * [earnings](https://github.com/stan-dev/stan/blob/feature/ARM/src/models/ARM/Ch.6/earnings1.stan): generalized linear model with logit link function and two predictors  
```
glm (earn_pos ~ height + male, family=binomial(link="logit"))
```

   * [earnings](https://github.com/stan-dev/stan/blob/feature/ARM/src/models/ARM/Ch.6/earnings2.stan): linear model with two predictors and log transformation 
```
lm (log_earn ~ height + male, subset=earn>0)
```

   * [earnings_vary_si](https://github.com/stan-dev/stan/blob/feature/ARM/src/models/ARM/Ch.13/earnings_vary_si.stan): multi-level linear model with group level predictors         
```
lmer (y ~ x (1 + x | ethn))
```

   * [earnings_latin_square](https://github.com/stan-dev/stan/blob/feature/ARM/src/models/ARM/Ch.13/earnings_latin_square.stan): non-nested multi-level linear model with group level predictors
```
lmer (y ~ x.centered + (1 + x.centered | eth) + (1 + x.centered | age) + (1 + x.centered | eth:age))
```

   * [earnings](https://github.com/stan-dev/stan/blob/feature/ARM/src/models/ARM/Ch.25/earnings.stan): linear model with ten predictors
```
lm (earnings ~ male + over65 + white + immig + educ_r + workmos + workhrs_top + any_ssi + any_welfare 
               + any_charity)
```

   * [earnings_pt1](https://github.com/stan-dev/stan/blob/feature/ARM/src/models/ARM/Ch.25/earnings.stan): logistic regression model with eight predictors
```
glm (earnings ~ male + over65 + white + immig + educ_r + any_ssi + any_welfare + any_charity,
     family=binomial(link="logit"))
```

   * [earnings_pt2](https://github.com/stan-dev/stan/blob/feature/ARM/src/models/ARM/Ch.25/earnings.stan): linear model with eight predictors
```
lm (earnings ~ male + over65 + white + immig + educ_r + any_ssi + any_welfare + any_charity)
```

   * [earnings2](https://github.com/stan-dev/stan/blob/feature/ARM/src/models/ARM/Ch.25/earnings2.stan): mlinear model with eleven predictors
```
lm (earnings ~ interest + male + over65 + white + immig + educ_r + workmos + workhrs_top + any_ssi 
               + any_welfare + any_charity)
```

***

#### Election88

   * [election88](https://github.com/stan-dev/stan/blob/feature/ARM/src/models/ARM/Ch.14/election88.stan): multi-level logistic regression model with group level predictors
```
lmer (y ~ black + female + (1 | state), family=binomial(link="logit"))
```
 
   * [election88_full](https://github.com/stan-dev/stan/blob/feature/ARM/src/models/ARM/Ch.14/election88_full.stan): multi-level logistic regression model with group level predictors
```
lmer (y ~ black + female + black:female + v.prev.full + (1 | age) + (1 | edu) + (1 | age.edu) 
          + (1 | state) + (1 | region.full), family=binomial(link="logit"))
```

   * [election88](https://github.com/stan-dev/stan/blob/feature/ARM/src/models/ARM/Ch.19/election88.stan): multi-level logistic regression model with redundant parameterization
```
lmer (y ~ female + black + female:black + (1 | age) + (1 | edu) + (1 | age_edu) + (1 | state), 
      family=binomial(link="logit"))
```

   * [election88_expansion](https://github.com/stan-dev/stan/blob/feature/ARM/src/models/ARM/Ch.19/election88_expansion.stan): multi-level logistic regression model with parameter expansion
```
lmer (y ~ female + black + female:black + (1 | age) + (1 | edu) + (1 | age_edu) + (1 | state), 
      family=binomial(link="logit"))
```

***

#### Electric

   * [electric](https://github.com/stan-dev/stan/blob/feature/ARM/src/models/ARM/Ch.9/electric_tr.stan): linear model with one predictor
```
lm (post_test ~ treatment)
```

   * [electric](https://github.com/stan-dev/stan/blob/feature/ARM/src/models/ARM/Ch.9/electric_trpre.stan): linear model with two predictors  
```
lm (post_test ~ pre_test + treatment)
```

   * [electric](https://github.com/stan-dev/stan/blob/feature/ARM/src/models/ARM/Ch.9/electric_inter.stan): linear model with two predictors and interaction  
```
lm (post_test ~ pre_test + treatment + pre_test:treatment)
```

   * [electric_1a](https://github.com/stan-dev/stan/blob/feature/ARM/src/models/ARM/Ch.23/electric_1a.stan): multi-level linear model with varying intercept and slope
```
lmer (y ~ 1 + (1 | pair) + (treatment | grade))
```

   * [electric_1b](https://github.com/stan-dev/stan/blob/feature/ARM/src/models/ARM/Ch.23/electric_1b.stan): multi-level linear model with varying intercept and slope
```
lmer (y ~ treatment + pre_test + (1 | pair))
```

   * [electric_1c](https://github.com/stan-dev/stan/blob/feature/ARM/src/models/ARM/Ch.23/electric_1c.stan): multi-level linear model with group level factors
```
lmer (y ~ 1 + (1 | pair) + (treatment | grade) + (pre_test | grade))
```

   * [electric](https://github.com/stan-dev/stan/blob/feature/ARM/src/models/ARM/Ch.23/electric.stan): multi-level linear model with varying intercept
```
lmer (y ~ treatment + (1 | pair))
```

***

#### Grades

   * [grades](https://github.com/stan-dev/stan/blob/feature/ARM/src/models/ARM/Ch.8/grades.stan): linear model with one predictor
```
lm (final ~ midterm)
```

***

#### HIV

   * [hiv](https://github.com/stan-dev/stan/blob/feature/ARM/src/models/ARM/Ch.20/hiv.stan): multi-level linear model with varying slope and intercept
```
lmer (y ~ time + (1 + time | person)
```

   * [hiv_inter](https://github.com/stan-dev/stan/blob/feature/ARM/src/models/ARM/Ch.20/hiv_inter.stan): multi-level linear model with interaction and varying slope and intercept
```
lmer (y ~ time:treatment + (1 + time | person)
```

#### Ideo

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
 
   * [ideo](https://github.com/stan-dev/stan/blob/feature/ARM/src/models/ARM/Ch.10/ideo_reparam.stan): linear model with two predictors and interaction  
```
lm (score1 ~ party + x + party:x, subset=incs)
```

***

#### Item Response

   * [item_response](https://github.com/stan-dev/stan/blob/feature/ARM/src/models/ARM/Ch.19/item_response.stan): multi-level logistic regression model with parameter expansion
```
lmer (y ~ a:g + (a:g | k,j) + (g:b | k), family=binomial(link="logit"))
```

***

#### Kid IQ

   * [kid_iq](https://github.com/stan-dev/stan/blob/feature/ARM/src/models/ARM/Ch.3/kidscore_momhs.stan): linear model with one predictor  
```
lm (kid_score ~ mom_hs)
```

   * [kid_iq](https://github.com/stan-dev/stan/blob/feature/ARM/src/models/ARM/Ch.3/kidscore_momiq.stan): linear model with one predictor  
```
lm (kid_score ~ mom_iq)
```

   * [kid_iq](https://github.com/stan-dev/stan/blob/feature/ARM/src/models/ARM/Ch.3/kidiq_multi_preds.stan): linear model with two predictors  
```
lm (kid_score ~ mom_hs + mom_iq)
```

   * [kid_iq](https://github.com/stan-dev/stan/blob/feature/ARM/src/models/ARM/Ch.3/kidiq_interaction.stan): linear model with two predictors and interaction  
```
lm (kid_score ~ mom_hs + mom_iq + mom_hs:mom_iq)
```

   * [kid_iq](https://github.com/stan-dev/stan/blob/feature/ARM/src/models/ARM/Ch.3/kidiq_validation.stan): linear model with two predictors  
```
lm (ppvt ~ hs + afqt)
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
   * [kid_iq](https://github.com/stan-dev/stan/blob/feature/ARM/src/models/ARM/Ch.4/kidscore_momwork.stan): linear model with one factor   
```
lm (kid_score ~ as.factor(mom_work))
```

***

#### Lightspeed

   * [lightspeed](https://github.com/stan-dev/stan/blob/feature/ARM/src/models/ARM/Ch.8/lightspeed.stan): linear model with no predictors
```
lm (y ~ 1)
```

***

#### Mesquite

   * [mesquite](https://github.com/stan-dev/stan/blob/feature/ARM/src/models/ARM/Ch.4/mesquite.stan): linear model with six predictors  
```
lm (weight~ diam1 + diam2 + canopy_height + total_height + density + group)
```

   * [mesquite](https://github.com/stan-dev/stan/blob/feature/ARM/src/models/ARM/Ch.4/mesquite_log.stan): linear model with six predictors and log transformation  
```
lm (log_weight~ log_diam1 + log_diam2 + log_canopy_height + log_total_height + log_density + group)
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

***

#### NES

   * [nes](https://github.com/stan-dev/stan/blob/feature/ARM/src/models/ARM/Ch.4/nes.stan): linear model with eight predictors    
```
lm (partyid7 ~ real_ideo + race_adj + age30_44 + age45_64 + age65up + educ1 + gender + income)
```

   * [nes](https://github.com/stan-dev/stan/blob/feature/ARM/src/models/ARM/Ch.5/nes_logit.stan): generalized linear model with  logit link function and one predictor  
```
glm (vote ~ income, family=binomial(link="logit"))
```

***

#### Pilots
 
   * [pilots](https://github.com/stan-dev/stan/blob/feature/ARM/src/models/ARM/Ch.13/pilots.stan): non-nested multi-level linear model with group level predictors         
```
lmer (y ~ 1 + (1 | group.id) (1 | scenario.id))
```
 
   * [pilots](https://github.com/stan-dev/stan/blob/feature/ARM/src/models/ARM/Ch.19/pilots.stan): multi-level linear model with varying intercept and redundant parameterization
```
lmer (y ~ 1 + (1 | treatment) + (1 | airport))
```

   * [pilots_expansion](https://github.com/stan-dev/stan/blob/feature/ARM/src/models/ARM/Ch.19/pilots_expansion.stan): multi-level linear model with varying intercept and parameter expansion
```
lmer (y ~ 1 + (1 | treatment) + (1 | airport))
```

***

#### Radon

   * [radon_intercept](https://github.com/stan-dev/stan/blob/feature/ARM/src/models/ARM/Ch.12/radon_intercept.stan): multi-level linear model with varying intercept        
```
lmer (y ~ 1 + (1 | county))
```

   * [radon_complete_pool](https://github.com/stan-dev/stan/blob/feature/ARM/src/models/ARM/Ch.12/radon_complete_pool.stan): multi-level linear model with complete pooling          
```
lm (y ~ x)
```

   * [radon_no_pool](https://github.com/stan-dev/stan/blob/feature/ARM/src/models/ARM/Ch.12/radon_no_pool.stan): multi-level linear model with no pooling        
```
lmer (y ~ x + (1 | county))
```

   * [radon_group](https://github.com/stan-dev/stan/blob/feature/ARM/src/models/ARM/Ch.12/radon_group.stan): multi-level linear model with group level predictor and individual level predictors              
```
lmer (y ~ x + u + (1 | county))
```

   * [radon_vary_si](https://github.com/stan-dev/stan/blob/feature/ARM/src/models/ARM/Ch.13/radon_vary_si.stan): multi-level linear model with group level predictors         
```
lmer (y ~ x (1 + x | county))
```

   * [radon_inter_vary](https://github.com/stan-dev/stan/blob/feature/ARM/src/models/ARM/Ch.13/radon_inter_vary.stan): multi-level linear model with group level predictors         
```
lmer (y ~ x + u.full + x:u.full + (1 + x | county))
```

   * [radon_redundant](https://github.com/stan-dev/stan/blob/feature/ARM/src/models/ARM/Ch.19/radon_redundant.stan): multi-level liner model with varying intercept and redundant parameterization
```
lmer (y ~ 1 + (1 | county))
```

   * [radon_vary_intercept_a](https://github.com/stan-dev/stan/blob/feature/ARM/src/models/ARM/Ch.21/radon_vary_intercept_a.stan): multi-level linear model with varying intercept set up to calculate pooling factors
```
lmer (y ~ x + (1 | county))
```

   * [radon_vary_intercept_b](https://github.com/stan-dev/stan/blob/feature/ARM/src/models/ARM/Ch.21/radon_vary_intercept_b.stan): multi-level linear model with varying intercept set up to calculate pooling factors
```
lmer (y ~ x + (1 | county))
```

   * [anova_radon_nopred](https://github.com/stan-dev/stan/blob/feature/ARM/src/models/ARM/Ch.22/anova_radon_nopred.stan): multi-level linear model with varying intercept and set up for ANOVA
```
lmer (y ~ 1 + (1 | county))
```

   * [radon_vary_intercept_floor](https://github.com/stan-dev/stan/blob/feature/ARM/src/models/ARM/Ch.21/radon_vary_intercept_floor.stan): multi-level linear model with varying intercept
```
lmer (y ~ u + x + (1 | county))
```

   * [radon_vary_intercept_floor2](https://github.com/stan-dev/stan/blob/feature/ARM/src/models/ARM/Ch.21/radon_vary_intercept_floor2.stan): multi-level linear model with varying intercept
```
lmer (y ~ u + x + x_mean + (1 | county))
```

   * [radon_vary_intercept_nofloor](https://github.com/stan-dev/stan/blob/feature/ARM/src/models/ARM/Ch.21/radon_vary_intercept_nofloor.stan): multi-level linear model with varying intercept
```
lmer (y ~ u + (1 | county))
```

***

#### Roaches

   * [roaches](https://github.com/stan-dev/stan/blob/feature/ARM/src/models/ARM/Ch.8/roaches.stan): poisson regression model with exposure and three predictors
```
glm (y ~ roach1 + treatment + senior, family=poisson, offset=log(exposure2))
```

   * [roaches_overdispersion](https://github.com/stan-dev/stan/blob/feature/ARM/src/models/ARM/Ch.8/roaches_overdispersion.stan): poisson overdispersion regression model with exposure and three predictors
```
glm(y ~ roach1 + treatment + senior, family=quasipoisson, offset=log(exposure2))
```

*** 

#### Sesame

   * [sesame](https://github.com/stan-dev/stan/blob/feature/ARM/src/models/ARM/Ch.10/sesame_one_pred_a.stan): linear model with one predictor  
```
lm (watched ~ encouraged)
```

   * [sesame](https://github.com/stan-dev/stan/blob/feature/ARM/src/models/ARM/Ch.10/sesame_one_pred_b.stan): linear model with one predictor  
```
lm (y ~ encouraged)
```

   * [sesame](https://github.com/stan-dev/stan/blob/feature/ARM/src/models/ARM/Ch.10/sesame_one_pred_2b.stan): linear model with one predictor  
```
lm (y ~ watched_hat)
```

   * [sesame](https://github.com/stan-dev/stan/blob/feature/ARM/src/models/ARM/Ch.10/sesame_multi_preds_3a.stan): linear model with three predictors and one factor  
```
lm (watched ~ encouraged + pretest + as.factor(site) + setting)
```

   * [sesame](https://github.com/stan-dev/stan/blob/feature/ARM/src/models/ARM/Ch.10/sesame_multi_preds_3b.stan): linear model with three predictors and one factor  
```
lm (y ~ watched_hat + pretest + as.factor(site) + setting)
```

   * [sesame_street1](https://github.com/stan-dev/stan/blob/feature/ARM/src/models/ARM/Ch.23/sesame_street1.stan): multi-level linear model using multivariate normal

   * [sesame_street2](https://github.com/stan-dev/stan/blob/feature/ARM/src/models/ARM/Ch.23/sesame_street2.stan): multi-level linear model using multivariate normal


***

#### Statistical Calculations

   * [multiple_comparisons](https://github.com/stan-dev/stan/blob/feature/ARM/src/models/ARM/Ch.21/multiple_comparison.stan): multi-level linear model that serves as a multiple comparisons example
```
lmer (y ~ theta (theta | j))
```

   * [r_sqr](https://github.com/stan-dev/stan/blob/feature/ARM/src/models/ARM/Ch.21/r_sqr.stan): multi-level linear model with appropriate calculations for R^2
```
lmer (y ~ 1 + (1 + x | county))
```

   * [finite_populations](https://github.com/stan-dev/stan/blob/feature/ARM/src/models/ARM/Ch.21/finite_populations.stan): linear model with appropriate calculations for calculating the standard deviation of a finite population
```
lm (g ~ u_1 + u)
```

***

#### Unemployment

   * [unemployment](https://github.com/stan-dev/stan/blob/feature/ARM/src/models/ARM/Ch.8/unemployment.stan): linear model with one predictor  
```
lm (y ~ y_lag)
```

***

#### Wells

   * [wells](https://github.com/stan-dev/stan/blob/feature/ARM/src/models/ARM/Ch.5/wells_dist.stan): generalized linear model with logit link function and one predictor  
```
glm (switc ~ dist, family=binomial(link="logit"))
```

   * [wells](https://github.com/stan-dev/stan/blob/feature/ARM/src/models/ARM/Ch.5/wells_dist100.stan): generalized linear model with logit link function and one predictor  
```
glm (switc ~ dist100, family=binomial(link="logit"))
```

   * [wells](https://github.com/stan-dev/stan/blob/feature/ARM/src/models/ARM/Ch.5/wells_interaction.stan): generalized linear model with logit link function and two predictors and interaction  
```
glm (switc ~ dist100 + arsenic + dist100:arsenic, family=binomial(link="logit"))
```

   * [wells](https://github.com/stan-dev/stan/blob/feature/ARM/src/models/ARM/Ch.5/wells_interaction_c.stan): generalized linear model with logit link function with two predictors and interaction centered using mean   
```
glm (switc ~ c_dist100 + c_arsenic + c_dist100:c_arsenic, family=binomial(link="logit"))
```

   * [wells](https://github.com/stan-dev/stan/blob/feature/ARM/src/models/ARM/Ch.5/wells_daae_c.stan): generalized linear model with logit link function and four predictors and interaction centered using mean  
```
glm (switc ~ c_dist100 + c_arsenic + c_dist100:c_arsenic + assoc + educ4, family=binomial(link="logit"))
```

   * [wells](https://github.com/stan-dev/stan/blob/feature/ARM/src/models/ARM/Ch.5/wells_dae_c.stan): generalized linear model with logit link function and three predictors and interaction centered using mean  
```
glm (switc ~ c_dist100 + c_arsenic + c_dist100:c_arsenic + educ4, family=binomial(link="logit"))
```

   * [wells](https://github.com/stan-dev/stan/blob/feature/ARM/src/models/ARM/Ch.5/wells_dae_inter_c.stan): generalized linear model with logit link function and three predictors and interaction centered using mean  
```
glm (switc ~ c_dist100 + c_arsenic + c_educ4 + c_dist100:c_arsenic + c_dist100:c_educ4 + c_arsenic:c_educ4, 
     family=binomial(link="logit"))
```

   * [wells](https://github.com/stan-dev/stan/blob/feature/ARM/src/models/ARM/Ch.5/wells_predicted.stan): generalized linear model with logit link function with three predictors and interaction with log transform and centered using mean   
```
glm (switc ~ c_dist100 + c_log_arsenic + c_educ4 + c_dist100:c_log_arsenic + c_dist100:c_educ4     
             + c_log_arsenic:c_educ4, 
     family=binomial(link="logit"))
```

   * [wells](https://github.com/stan-dev/stan/blob/feature/ARM/src/models/ARM/Ch.5/wells_predicted_log.stan): generalized linear model with logit link function with three predictors and interaction with log transform and centered using mean  
```
glm (switc ~ dist100 + log_arsenic + educ4 + dist100:log_arsenic + dist100:educ4 + log_arsenic:educ4, 
     family=binomial(link="logit"))
```

   * [wells](https://github.com/stan-dev/stan/blob/feature/ARM/src/models/ARM/Ch.5/wells_dae.stan): generalized linear model with logit link function and three predictors  
```
glm (switc ~ dist100 + arsenic + educ4, family=binomial(link="logit"))
```

   * [wells](https://github.com/stan-dev/stan/blob/feature/ARM/src/models/ARM/Ch.5/wells_dae_inter.stan): generalized linear model with logit link function and three predictors with interaction  
```
glm (switc ~ dist100 + arsenic + educ4 + dist100:arsenic, family=binomial(link="logit"))
```

   * [wells](https://github.com/stan-dev/stan/blob/feature/ARM/src/models/ARM/Ch.6/wells_probit.stan): generalized linear model with probit link function and one predictor  
```
glm (switc ~ dist100, family=binomial(link="probit"))
```
