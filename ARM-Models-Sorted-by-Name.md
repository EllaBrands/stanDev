Each chapter has a README file on github that you can view by clicking on the chapter link and scrolling down (past the files). THe README file contains information about the data contained in the chapter (particularly, what each variable represents) and sorts the models in the chapter by type.

If there is a * next to a model name, then the model DOES NOT currently work with [RStanARM](https://github.com/stan-dev/rstanarm). Furthermore, multilevel models are not currently supported by RStanARM, but all other models below (without a *) are supported.

#### kid iq

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

#### earnings

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

***

#### mesquite

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

#### nes

   * [nes](https://github.com/stan-dev/stan/blob/feature/ARM/src/models/ARM/Ch.4/nes.stan): linear model with eight predictors    
```
lm (partyid7 ~ real_ideo + race_adj + age30_44 + age45_64 + age65up + educ1 + gender + income)
```

   * [nes](https://github.com/stan-dev/stan/blob/feature/ARM/src/models/ARM/Ch.5/nes.stan): generalized linear model with  logit link function and one predictor  
```
glm (vote ~ income, family=binomial(link="logit"))
```

***

#### wells

   * [wells](https://github.com/stan-dev/stan/blob/feature/ARM/src/models/ARM/Ch.5/wells_one_pred.stan): generalized linear model with logit link function and one predictor  
```
glm (switc ~ dist, family=binomial(link="logit"))
```

   * [wells](https://github.com/stan-dev/stan/blob/feature/ARM/src/models/ARM/Ch.5/wells_one_pred_scale.stan): generalized linear model with logit link function and one predictor  
```
glm (switc ~ dist100, family=binomial(link="logit"))
```

   * [wells](https://github.com/stan-dev/stan/blob/feature/ARM/src/models/ARM/Ch.5/wells_interactions.stan): generalized linear model with logit link function and two predictors and interaction  
```
glm (switc ~ dist100 + arsenic + dist100:arsenic, family=binomial(link="logit"))
```

   * [wells](https://github.com/stan-dev/stan/blob/feature/ARM/src/models/ARM/Ch.5/wells_interactions_center.stan): generalized linear model with logit link function with two predictors and interaction centered using mean   
```
glm (switc ~ c_dist100 + c_arsenic + c_dist100:c_arsenic, family=binomial(link="logit"))
```

   * [wells](https://github.com/stan-dev/stan/blob/feature/ARM/src/models/ARM/Ch.5/wells_community.stan): generalized linear model with logit link function and four predictors and interaction centered using mean  
```
glm (switc ~ c_dist100 + c_arsenic + c_dist100:c_arsenic + assoc + educ4, family=binomial(link="logit"))
```

   * [wells](https://github.com/stan-dev/stan/blob/feature/ARM/src/models/ARM/Ch.5/wells_social.stan): generalized linear model with logit link function and three predictors and interaction centered using mean  
```
glm (switc ~ c_dist100 + c_arsenic + c_dist100:c_arsenic + educ4, family=binomial(link="logit"))
```

   * [wells](https://github.com/stan-dev/stan/blob/feature/ARM/src/models/ARM/Ch.5/wells_interactions_center_educ.stan): generalized linear model with logit link function and three predictors and interaction centered using mean  
```
glm (switc ~ c_dist100 + c_arsenic + c_educ4 + c_dist100:c_arsenic + c_dist100:c_educ4 + c_arsenic:c_educ4, 
     family=binomial(link="logit"))
```

   * [wells](https://github.com/stan-dev/stan/blob/feature/ARM/src/models/ARM/Ch.5/wells_log_transform.stan): generalized linear model with logit link function with three predictors and interaction with log transform and centered using mean   
```
glm (switc ~ c_dist100 + c_log_arsenic + c_educ4 + c_dist100:c_log_arsenic + c_dist100:c_educ4     
             + c_log_arsenic:c_educ4, 
     family=binomial(link="logit"))
```

   * [wells](https://github.com/stan-dev/stan/blob/feature/ARM/src/models/ARM/Ch.5/wells_log_transform2.stan): generalized linear model with logit link function with three predictors and interaction with log transform and centered using mean  
```
glm (switc ~ dist100 + log_arsenic + educ4 + dist100:log_arsenic + dist100:educ4 + log_arsenic:educ4, 
     family=binomial(link="logit"))
```

   * [wells](https://github.com/stan-dev/stan/blob/feature/ARM/src/models/ARM/Ch.5/wells_edu.stan): generalized linear model with logit link function and three predictors  
```
glm (switc ~ dist100 + arsenic + educ4, family=binomial(link="logit"))
```

   * [wells](https://github.com/stan-dev/stan/blob/feature/ARM/src/models/ARM/Ch.5/wells_all.stan): generalized linear model with logit link function and three predictors with interaction  
```
glm (switc ~ dist100 + arsenic + educ4 + dist100:arsenic, family=binomial(link="logit"))
```

   * [wells](https://github.com/stan-dev/stan/blob/feature/ARM/src/models/ARM/Ch.6/wells_probit.stan): generalized linear model with probit link function and one predictor  
```
glm (switc ~ dist100, family=binomial(link="probit"))
```

***

#### congress

   * [congress](https://github.com/stan-dev/stan/blob/feature/ARM/src/models/ARM/Ch.7/congress.stan): linear model with two predictors  
```
lm (vote_88 ~ vote_86 + incumbency_88)
```
 
*** grades

   * [grades](https://github.com/stan-dev/stan/blob/feature/ARM/src/models/ARM/Ch.8/grades.stan): linear model with one predictor
```
lm (final ~ midterm)
```

***

#### lightspeed

   * [lightspeed](https://github.com/stan-dev/stan/blob/feature/ARM/src/models/ARM/Ch.8/lightspeed.stan): linear model with no predictors
```
lm (y ~ 1)
```

***

#### roaches

   * [roaches](https://github.com/stan-dev/stan/blob/feature/ARM/src/models/ARM/Ch.8/roaches.stan): poisson regression model with exposure and three predictors
```
glm (y ~ roach1 + treatment + senior, family=poisson, offset=log(exposure2))
```

   * [roaches_overdispersion](https://github.com/stan-dev/stan/blob/feature/ARM/src/models/ARM/Ch.8/roaches_overdispersion.stan): poisson overdispersion regression model with exposure and three predictors
```
glm(y ~ roach1 + treatment + senior, family=quasipoisson, offset=log(exposure2))
```

*** 

#### unemployment

   * [unemployment](https://github.com/stan-dev/stan/blob/feature/ARM/src/models/ARM/Ch.8/unemployment.stan): linear model with one predictor  
```
lm (y ~ y_lag)
```

***

#### electric

   * [electric](https://github.com/stan-dev/stan/blob/feature/ARM/src/models/ARM/Ch.9/electric_one_pred.stan): linear model with one predictor
```
lm (post_test ~ treatment)
```

   * [electric](https://github.com/stan-dev/stan/blob/feature/ARM/src/models/ARM/Ch.9/electric_multi_preds.stan): linear model with two predictors  
```
lm (post_test ~ pre_test + treatment)
```

   * [electric](https://github.com/stan-dev/stan/blob/feature/ARM/src/models/ARM/Ch.9/electric_interactions.stan): linear model with two predictors and interaction  
```
lm (post_test ~ pre_test + treatment + pre_test:treatment)
```

***

#### ideo

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

#### sesame

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

***

#### radon

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

  


  * [13.5 Non-Nested Models](https://github.com/stan-dev/stan/blob/feature/ARM/src/models/ARM/Ch.13/13.5_Non-NestedModels.R)
 
   * [pilots](https://github.com/stan-dev/stan/blob/feature/ARM/src/models/ARM/Ch.13/pilots.stan): non-nested multi-level linear model with group level predictors         
```
lmer (y ~ 1 + (1 | group.id) (1 | scenario.id))
```
 
   * [earnings_latin_square](https://github.com/stan-dev/stan/blob/feature/ARM/src/models/ARM/Ch.13/earnings_latin_square.stan): non-nested multi-level linear model with group level predictors
```
lmer (y ~ x.centered + (1 + x.centered | eth) + (1 + x.centered | age) + (1 + x.centered | eth:age))
```

***

#### [Chapter 14 - Multilevel Logistic Regression](https://github.com/stan-dev/stan/tree/feature/ARM/src/models/ARM/Ch.14)
 
  * [14.1 State-Level Opinions From National Polls](https://github.com/stan-dev/stan/blob/feature/ARM/src/models/ARM/Ch.14/14.1_State-LevelOpinionsFromNationalPolls.R)

    * [elections88](https://github.com/stan-dev/stan/blob/feature/ARM/src/models/ARM/Ch.14/elections88.stan): multi-level logistic regression model with group level predictors
```
lmer (y ~ black + female + (1 | state), family=binomial(link="logit"))
```
 
    * [elections88_full](https://github.com/stan-dev/stan/blob/feature/ARM/src/models/ARM/Ch.14/elections88_full.stan): multi-level logistic regression model with group level predictors
```
lmer (y ~ black + female + black:female + v.prev.full + (1 | age) + (1 | edu) + (1 | age.edu) 
          + (1 | state) + (1 | region.full), family=binomial(link="logit"))
```

***

#### [Chapter 15 - Multilevel Generalized Linear Models](https://github.com/stan-dev/stan/tree/feature/ARM/src/models/ARM/Ch.15)

***

#### [Chapter 19 - Debugging and Speeding Convergence](https://github.com/stan-dev/stan/tree/feature/ARM/src/models/ARM/Ch.19)

  * [19.2 General Methods for Reducing Computational Requirements] (https://github.com/stan-dev/stan/blob/feature/ARM/src/models/ARM/Ch.19/19.2_GeneralMethodsForReducingComputationalRequirements.R)
  
  * [19.3 Simple Linear Transformations](https://github.com/stan-dev/stan/blob/feature/ARM/src/models/ARM/Ch.19/19.3_SimpleLinearTransformations.R)

  * [19.4 Redundant Parameters & Intentionally Non-identifiable Models](https://github.com/stan-dev/stan/blob/feature/ARM/src/models/ARM/Ch.19/19.4_RedundantParameters%26IntentionallyNonidentifiableModels.R)

    * [radon](https://github.com/stan-dev/stan/blob/feature/ARM/src/models/ARM/Ch.19/radon.stan): multi-level liner model with varying intercept
```
lmer (y ~ 1 + (1 | county))
```

    * [radon_redundant](https://github.com/stan-dev/stan/blob/feature/ARM/src/models/ARM/Ch.19/radon_redundant.stan): multi-level liner model with varying intercept and redundant parameterization
```
lmer (y ~ 1 + (1 | county))
```

    * [pilots](https://github.com/stan-dev/stan/blob/feature/ARM/src/models/ARM/Ch.19/pilots.stan): multi-level linear model with varying intercept and redundant parameterization
```
lmer (y ~ 1 + (1 | treatment) + (1 | airport))
```

    * [election88](https://github.com/stan-dev/stan/blob/feature/ARM/src/models/ARM/Ch.19/election88.stan): multi-level logistic regression model with redundant parameterization
```
lmer (y ~ female + black + female:black + (1 | age) + (1 | edu) + (1 | age_edu) + (1 | state), 
      family=binomial(link="logit"))
```

  * [19.5 Parameter Expansion](https://github.com/stan-dev/stan/blob/feature/ARM/src/models/ARM/Ch.19/19.5_ParameterExpansion.R)

    * [pilots_expansion](https://github.com/stan-dev/stan/blob/feature/ARM/src/models/ARM/Ch.19/pilots_expansion.stan): multi-level linear model with varying intercept and parameter expansion
```
lmer (y ~ 1 + (1 | treatment) + (1 | airport))
```

    * [election88_expansion](https://github.com/stan-dev/stan/blob/feature/ARM/src/models/ARM/Ch.19/election88_expansion.stan): multi-level logistic regression model with parameter expansion
```
lmer (y ~ female + black + female:black + (1 | age) + (1 | edu) + (1 | age_edu) + (1 | state), 
      family=binomial(link="logit"))
```

    * [item_response](https://github.com/stan-dev/stan/blob/feature/ARM/src/models/ARM/Ch.19/item_response.stan): multi-level logistic regression model with parameter expansion
```
lmer (y ~ a:g + (a:g | k,j) + (g:b | k), family=binomial(link="logit"))
```

  * [19.6 Using Redundant Parameters for Modeling](https://github.com/stan-dev/stan/blob/feature/ARM/src/models/ARM/Ch.19/19.6_UsingRedundantParametersForModeling.R)

    * [8_schools](https://github.com/stan-dev/stan/blob/feature/ARM/src/models/ARM/Ch.19/schools.stan): multi-level linear model with redundant parameterization

***

#### [Chapter 20 - Sample Size and Power Calculations](https://github.com/stan-dev/stan/tree/feature/ARM/src/models/ARM/Ch.20)

  * [20.5 Multilevel Power Calculation Using Fake-Data Simulation](https://github.com/stan-dev/stan/blob/feature/ARM/src/models/ARM/Ch.20/20.5_MultilevelPowerCalculationUsingFake-DataSimulation.R)

    * [hiv](https://github.com/stan-dev/stan/blob/feature/ARM/src/models/ARM/Ch.20/hiv.stan): multi-level linear model with varying slope and intercept
```
lmer (y ~ time + (1 + time | person)
```

    * [hiv_inter](https://github.com/stan-dev/stan/blob/feature/ARM/src/models/ARM/Ch.20/hiv_inter.stan): multi-level linear model with interaction and varying slope and intercept
```
lmer (y ~ time:treatment + (1 + time | person)
```

***

#### [Chapter 21 - Understanding and Summarizing the Fitted Models](https://github.com/stan-dev/stan/tree/feature/ARM/src/models/ARM/Ch.21)

  * [21.2 Superpopulation & Finite-Population Variances](https://github.com/stan-dev/stan/blob/feature/ARM/src/models/ARM/Ch.21/21.2_Superpopulation%26Finite-PopulationVariances.R)

    * [finite_populations](https://github.com/stan-dev/stan/blob/feature/ARM/src/models/ARM/Ch.21/finite_populations.stan): linear model with appropriate calculations for calculating the standard deviation of a finite population
```
lm (g ~ u_1 + u)
```

  * [21.3 Contrasts & Comparisons of Multilevel Coefficients](https://github.com/stan-dev/stan/blob/feature/ARM/src/models/ARM/Ch.21/21.3_Contrasts%26ComparisonsOfMultilevelCoefficients.R)

  * [21.5 R^2 & Explained Variance](https://github.com/stan-dev/stan/blob/feature/ARM/src/models/ARM/Ch.21/21.5_R%5E2%26ExplainedVariance.R)

    * [r_sqr](https://github.com/stan-dev/stan/blob/feature/ARM/src/models/ARM/Ch.21/r_sqr.stan): multi-level linear model with appropriate calculations for R^2
```
lmer (y ~ 1 + (1 + x | county))
```

  * [21.6 Summarizing the Amount of Partial Pooling](https://github.com/stan-dev/stan/blob/feature/ARM/src/models/ARM/Ch.21/21.6_SummarizingtheAmmountofPartialPooling.R)

    * [radon_vary_intercept_a](https://github.com/stan-dev/stan/blob/feature/ARM/src/models/ARM/Ch.21/radon_vary_intercept_a.stan): multi-level linear model with varying intercept set up to calculate pooling factors
```
lmer (y ~ x + (1 | county))
```

    * [radon_vary_intercept_b](https://github.com/stan-dev/stan/blob/feature/ARM/src/models/ARM/Ch.21/radon_vary_intercept_b.stan): multi-level linear model with varying intercept set up to calculate pooling factors
```
lmer (y ~ x + (1 | county))
```

    * [radon_vary_intercept_c](https://github.com/stan-dev/stan/blob/feature/ARM/src/models/ARM/Ch.21/radon_vary_intercept_c.stan): multi-level linear model with varying intercept
```
lmer (y ~ x + u + (1 | county))
```

  * [21.7 Adding a Predictor can Increase Residual Variance](https://github.com/stan-dev/stan/blob/feature/ARM/src/models/ARM/Ch.21/21.7_AddingAPredictorCanIncreaseResidualVariance.R)

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

  * 21.8 Multiple Comparisons and Statistical Significance

    * [multiple_comparisons](https://github.com/stan-dev/stan/blob/feature/ARM/src/models/ARM/Ch.21/multiple_comparison.stan): multi-level linear model that serves as a multiple comparisons example
```
lmer (y ~ theta (theta | j))
```

***

#### [Chapter 22 - Analysis of Variance](https://github.com/stan-dev/stan/tree/feature/ARM/src/models/ARM/Ch.22)

  * [22.1 Classical ANOVA](https://github.com/stan-dev/stan/blob/feature/ARM/src/models/ARM/Ch.22/22.1_ClassicalANOVA.R)

  * [22.4 Doing ANOVA Using MLM](https://github.com/stan-dev/stan/blob/feature/ARM/src/models/ARM/Ch.22/22.4_DoingANOVUsingMLM.R)

    * [anova_radon_nopred](https://github.com/stan-dev/stan/blob/feature/ARM/src/models/ARM/Ch.22/anova_radon_nopred.stan): multi-level linear model with varying intercept and set up for ANOVA
```
lmer (y ~ 1 + (1 | county))
```

***

#### [Chapter 23 - Casual Inference Using Multilevel Models](https://github.com/stan-dev/stan/tree/feature/ARM/src/models/ARM/Ch.23)

  * [23.1 Multilevel Aspects of Data Collection](https://github.com/stan-dev/stan/blob/feature/ARM/src/models/ARM/Ch.23/23.1_MultilevelAspectsofDataCollection.R)

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

    * [electric_one_pred](https://github.com/stan-dev/stan/blob/feature/ARM/src/models/ARM/Ch.23/electric_one_pred.stan): linear model with one predictor
```
lm (post_test ~ treatment)
```

    * [electric_multi_preds](https://github.com/stan-dev/stan/blob/feature/ARM/src/models/ARM/Ch.23/electric_multi_preds.stan): linear model with two predictors
```
lm (post_test ~ treatment + pre_test)
```

  * 23.3 Treatments Applied at Different Levels

    * [electric](https://github.com/stan-dev/stan/blob/feature/ARM/src/models/ARM/Ch.23/electric.stan): multi-level linear model with varying intercept
```
lmer (y ~ treatment + (1 | pair))
```

  * [23.4 Instrumental Variables & Multilevel Modeling](https://github.com/stan-dev/stan/blob/feature/ARM/src/models/ARM/Ch.23/23.4_InstrumentalVariables%26MultilevelModeling.R)

    * [sesame_street1](https://github.com/stan-dev/stan/blob/feature/ARM/src/models/ARM/Ch.23/sesame_street1.stan): multi-level linear model using multivariate normal

    * [sesame_street2](https://github.com/stan-dev/stan/blob/feature/ARM/src/models/ARM/Ch.23/sesame_street2.stan): multi-level linear model using multivariate normal

***

#### [Chapter 24 - Model Checking and Comparison](https://github.com/stan-dev/stan/tree/feature/ARM/src/models/ARM/Ch.24)

  * [24.2 Behavioral Learning Experiments](https://github.com/stan-dev/stan/blob/feature/ARM/src/models/ARM/Ch.24/24.2_BehavioralLearningExperiment.R)

    * [dogs](https://github.com/stan-dev/stan/blob/feature/ARM/src/models/ARM/Ch.24/dogs.stan): multi-level logit regression model

    * [dogs_log](https://github.com/stan-dev/stan/blob/feature/ARM/src/models/ARM/Ch.24/dogs_log.stan): multi-level model using binomial distribution

    * [dogs_check](https://github.com/stan-dev/stan/blob/feature/ARM/src/models/ARM/Ch.24/dogs_check.stan): multi-level model using binomial distribution

***

#### [Chapter 25 - Missing-Data Imuptation](https://github.com/stan-dev/stan/tree/feature/ARM/src/models/ARM/Ch.25)

  * [25.0 Missing Data in R and Bugs](https://github.com/stan-dev/stan/blob/feature/ARM/src/models/ARM/Ch.25/25.0_MissingDataInRandBugs.R)

  * [25.4 Random Imputation of a Single Variable](https://github.com/stan-dev/stan/blob/feature/ARM/src/models/ARM/Ch.25/25.4_RadomImputationofaSingleVariable.R)

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

  * [25.5 Imputation of Several Missing variables](https://github.com/stan-dev/stan/blob/feature/ARM/src/models/ARM/Ch.25/25.5_ImputationofSeveralMissingVariables.R)

    * [earnings2](https://github.com/stan-dev/stan/blob/feature/ARM/src/models/ARM/Ch.25/earnings2.stan): mlinear model with eleven predictors
```
lm (earnings ~ interest + male + over65 + white + immig + educ_r + workmos + workhrs_top + any_ssi 
               + any_welfare + any_charity)
```

***

THE BELOW CHAPTERS HAVE NOT BEEN ADDED YET.

#### [Chapter 16 - Multilevel Modeling in Bugs and R: the Basics](https://github.com/stan-dev/stan/tree/feature/ARM/src/models/ARM/Ch.16)

#### [Chapter 17 - Fitting Multilevel Linear and Generalized Linear Models in Bugs and R](https://github.com/stan-dev/stan/tree/feature/ARM/src/models/ARM/Ch.17)

#### [Chapter 18 - Likelihood and Bayesian Inference and Computation](https://github.com/stan-dev/stan/tree/feature/ARM/src/models/ARM/Ch.18)
 