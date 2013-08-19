Each chapter has a README file on github that you can view by clicking on the chapter link and scrolling down (past the files). The README file contains information about the data contained in the chapter (particularly, what each variable represents) and sorts the models in the chapter by type.

If there is a * next to a model name, then the model DOES NOT currently work with [RStanARM](https://github.com/stan-dev/rstanarm). Furthermore, multilevel models are not currently supported by RStanARM, but all other models below (without a *) are supported.

* [Chapter 2 - Concepts and Methods from Basic Probability and Statistics](https://github.com/stan-dev/stan/wiki/ARM-Models-Sorted-by-Chapter#chapter-2---concepts-and-methods-from-basic-probability-and-statistics)
* [Chapter 3 - Linear Regression: the Basics](https://github.com/stan-dev/stan/wiki/ARM-Models-Sorted-by-Chapter#chapter-3---linear-regression-the-basics)
* [Chapter 4 - Linear Regression: Before and After Fitting the Model](https://github.com/stan-dev/stan/wiki/ARM-Models-Sorted-by-Chapter#chapter-4---linear-regression-before-and-after-fitting-the-model)
* [Chapter 5 - Logistic Regression](https://github.com/stan-dev/stan/wiki/ARM-Models-Sorted-by-Chapter#chapter-5---logistic-regression)
* [Chapter 6 - Generalized Linear Models](https://github.com/stan-dev/stan/wiki/ARM-Models-Sorted-by-Chapter#chapter-6---generalized-linear-models)
* [Chapter 7 - Simulation of Probability Models and Statistical Inference](https://github.com/stan-dev/stan/wiki/ARM-Models-Sorted-by-Chapter#chapter-7---simulation-of-probability-models-and-statistical-inferences)
* [Chapter 8 - Simulation for Checking Statistical Procedures and Model Fits](https://github.com/stan-dev/stan/wiki/ARM-Models-Sorted-by-Chapter#chapter-8---simulation-for-checking-statistical-procedures-and-model-fits)
* [Chapter 9 - Casual Inference Using Regression on the Treatment Variable](https://github.com/stan-dev/stan/wiki/ARM-Models-Sorted-by-Chapter#chapter-9---casual-inference-using-regression-on-the-treatment-variable)
* [Chapter 10 - Casual Inference Using More Advanced Models](https://github.com/stan-dev/stan/wiki/ARM-Models-Sorted-by-Chapter#chapter-10---casual-inference-using-more-advanced-models)
* [Chapter 11 - Multilevel Structures](https://github.com/stan-dev/stan/wiki/ARM-Models-Sorted-by-Chapter#chapter-11---multilevel-structures)
* [Chapter 12 - Multilevel Linear Models: the Basics](https://github.com/stan-dev/stan/wiki/ARM-Models-Sorted-by-Chapter#chapter-12---multilevel-linear-models-the-basics)
* [Chapter 13 - Multilevel Linear Models: Varying Slopes, Non-Nested Models, and Other Complexities](https://github.com/stan-dev/stan/wiki/ARM-Models-Sorted-by-Chapter#chapter-13---multilevel-linear-models-varying-slopes-non-nested-models-and-other-complexities)
* [Chapter 14 - Multilevel Logistic Regression](https://github.com/stan-dev/stan/wiki/ARM-Models-Sorted-by-Chapter#chapter-14---multilevel-logistic-regression)
* [Chapter 15 - Multilevel Generalized Linear Models](https://github.com/stan-dev/stan/wiki/ARM-Models-Sorted-by-Chapter#chapter-15---multilevel-generalized-linear-models)
* [Chapter 16 - Multilevel Modeling in Bugs and R: the Basics](https://github.com/stan-dev/stan/wiki/ARM-Models-Sorted-by-Chapter#chapter-16---multilevel-modeling-in-bugs-and-r-the-basics)
* [Chapter 17 - Fitting Multilevel Linear and Generalized Linear Models in Bugs and R](https://github.com/stan-dev/stan/wiki/ARM-Models-Sorted-by-Chapter#chapter-17---fitting-multilevel-linear-and-generalized-linear-models-in-bugs-and-r)
* [Chapter 18 - Likelihood and Bayesian Inference and Computation](https://github.com/stan-dev/stan/wiki/ARM-Models-Sorted-by-Chapter#chapter-18---likelihood-and-bayesian-inference-and-computation)
* [Chapter 19 - Debugging and Speeding Convergence](https://github.com/stan-dev/stan/wiki/ARM-Models-Sorted-by-Chapter#chapter-19---debugging-and-speeding-convergence)
* [Chapter 20 - Sample Size and Power Calculations](https://github.com/stan-dev/stan/wiki/ARM-Models-Sorted-by-Chapter#chapter-20---sample-size-and-power-calculations)
* [Chapter 21 - Understanding and Summarizing the Fitted Models](https://github.com/stan-dev/stan/wiki/ARM-Models-Sorted-by-Chapter#chapter-21---understanding-and-summarizing-the-fitted-models)
* [Chapter 22 - Analysis of Variance](https://github.com/stan-dev/stan/wiki/ARM-Models-Sorted-by-Chapter#chapter-22---analysis-of-variance)
* [Chapter 23 - Casual Inference Using Multilevel Models](https://github.com/stan-dev/stan/wiki/ARM-Models-Sorted-by-Chapter#chapter-23---casual-inference-using-multilevel-models)
* [Chapter 24 - Model Checking and Comparison](https://github.com/stan-dev/stan/wiki/ARM-Models-Sorted-by-Chapter#chapter-24---model-checking-and-comparison)
* [Chapter 25 - Missing-Data Imputation](https://github.com/stan-dev/stan/wiki/ARM-Models-Sorted-by-Chapter#chapter-25---missing-data-imuptation)


***

***

#### [Chapter 2 - Concepts and Methods from Basic Probability and Statistics](https://github.com/stan-dev/stan/tree/feature/ARM/src/models/ARM/Ch.2)

 * [2.3 CI](https://github.com/stan-dev/stan/blob/feature/ARM/src/models/ARM/Ch.2/2.3_CI.R)

 * [2.4 Hypothesis Testing - MISSING DATA](https://github.com/stan-dev/stan/blob/feature/ARM/src/models/ARM/Ch.2/2.4_HypothesisTesting.R)

 * [2.6 55,000 Residents - MISSING DATA](https://github.com/stan-dev/stan/blob/feature/ARM/src/models/ARM/Ch.2/2.6_55%2C000Residents.R)

***

#### [Chapter 3 - Linear Regression: the Basics](https://github.com/stan-dev/stan/tree/feature/ARM/src/models/ARM/Ch.3)

 * [3.1 One Predictor](https://github.com/stan-dev/stan/blob/feature/ARM/src/models/ARM/Ch.3/3.1_OnePredictor.R)

   * [kidscore_momhs](https://github.com/stan-dev/stan/blob/feature/ARM/src/models/ARM/Ch.3/kidscore_momhs.stan): linear model with one predictor  
```
lm (kid_score ~ mom_hs)
```

   * [kidscore_momiq](https://github.com/stan-dev/stan/blob/feature/ARM/src/models/ARM/Ch.3/kidscore_momiq.stan): linear model with one predictor  
```
lm (kid_score ~ mom_iq)
```

 * [3.2 Multiple Predictors](https://github.com/stan-dev/stan/blob/feature/ARM/src/models/ARM/Ch.3/3.2_MultiplePredictors.R)

   * [kidiq_multi_preds](https://github.com/stan-dev/stan/blob/feature/ARM/src/models/ARM/Ch.3/kidiq_multi_preds.stan): linear model with two predictors  
```
lm (kid_score ~ mom_hs + mom_iq)
```

 * [3.3 Interactions](https://github.com/stan-dev/stan/blob/feature/ARM/src/models/ARM/Ch.3/3.3_Interactions.R)

   * [kidiq_interaction](https://github.com/stan-dev/stan/blob/feature/ARM/src/models/ARM/Ch.3/kidiq_interaction.stan): linear model with two predictors and interaction  
```
lm (kid_score ~ mom_hs + mom_iq + mom_hs:mom_iq)
```

 * [3.4 Stat Inference](https://github.com/stan-dev/stan/blob/feature/ARM/src/models/ARM/Ch.3/3.4_StatInference.R)

   * [kidiq_multi_preds](https://github.com/stan-dev/stan/blob/feature/ARM/src/models/ARM/Ch.3/kidiq_multi_preds.stan): linear model with two predictors  
```
lm (kid_score ~ mom_hs + mom_iq)
```

 * [3.5 Graph Displays](https://github.com/stan-dev/stan/blob/feature/ARM/src/models/ARM/Ch.3/3.5_GraphDisplays.R)

   * [kidscore_momiq](https://github.com/stan-dev/stan/blob/feature/ARM/src/models/ARM/Ch.3/kidscore_momiq.stan): linear model with one predictor  
```
lm (kid_score ~ mom_iq)
```

   * [kidiq_multi_preds](https://github.com/stan-dev/stan/blob/feature/ARM/src/models/ARM/Ch.3/kidiq_multi_preds.stan): linear model with two predictors  
```
lm (kid_score ~ mom_hs + mom_iq)
```

   * [kidiq_interaction](https://github.com/stan-dev/stan/blob/feature/ARM/src/models/ARM/Ch.3/kidiq_interaction.stan): linear model with two predictors and interaction  
```
lm (kid_score ~ mom_hs + mom_iq + mom_hs:mom_iq)
```

 * [3.6 Diagnostics](https://github.com/stan-dev/stan/blob/feature/ARM/src/models/ARM/Ch.3/3.6_Diagnostics.R)

   * [kidscore_momiq](https://github.com/stan-dev/stan/blob/feature/ARM/src/models/ARM/Ch.3/kidscore_momiq.stan): linear model with one predictor  
```
lm (kid_score ~ mom_iq)
```

 * [3.7 Prediction](https://github.com/stan-dev/stan/blob/feature/ARM/src/models/ARM/Ch.3/3.7_Prediction.R)

   * [kidiq_multi_preds](https://github.com/stan-dev/stan/blob/feature/ARM/src/models/ARM/Ch.3/kidiq_multi_preds.stan): linear model with two predictors  
```
lm (kid_score ~ mom_hs + mom_iq)
```

   * [kidiq_validation](https://github.com/stan-dev/stan/blob/feature/ARM/src/models/ARM/Ch.3/kidiq_validation.stan): linear model with two predictors  
```
lm (ppvt ~ hs + afqt)
```

***

#### [Chapter 4 - Linear Regression: Before and After Fitting the Model](https://github.com/stan-dev/stan/tree/feature/ARM/src/models/ARM/Ch.4)

  * [4.1 Linear Transformations](https://github.com/stan-dev/stan/blob/feature/ARM/src/models/ARM/Ch.4/4.1_LinearTransformations.R)

   * [earn_height](https://github.com/stan-dev/stan/blob/feature/ARM/src/models/ARM/Ch.4/earn_height.stan): linear model with one predictor  
```
lm (earnings ~ height)
```

  * [4.2 Centering & Standardizing](https://github.com/stan-dev/stan/blob/feature/ARM/src/models/ARM/Ch.4/4.2_Centering%26Standardizing.R)

   * [kidiq_interaction](https://github.com/stan-dev/stan/blob/feature/ARM/src/models/ARM/Ch.4/kidiq_interaction.stan): linear model with two predictors and interaction  
```
lm (kid_score ~ mom_hs + mom_iq + mom_hs:mom_iq)
```

   * [kidiq_interaction_c](https://github.com/stan-dev/stan/blob/feature/ARM/src/models/ARM/Ch.4/kidiq_interaction_c.stan): linear model with two predictors and interaction centered using mean  
```
lm (kid_score ~ c_mom_hs + c_mom_iq + c_mom_hs:c_mom_iq)
```

   * [kidiq_interaction_c2](https://github.com/stan-dev/stan/blob/feature/ARM/src/models/ARM/Ch.4/kidiq_interaction_c2.stan): linear model with two predictors and interaction centered using conventional points  
```
lm (kid_score ~ c2_mom_hs + c2_mom_iq + c2_mom_hs:c2_mom_iq)
```

   * [kidiq_interaction_z](https://github.com/stan-dev/stan/blob/feature/ARM/src/models/ARM/Ch.4/kidiq_interaction_z.stan): linear model with two predictors and interaction centered using z-score  
```
lm (kid_score ~ z_mom_hs + z_mom_iq + z_mom_hs:z_mom_iq)
```

  * [4.4 Log Transformations](https://github.com/stan-dev/stan/blob/feature/ARM/src/models/ARM/Ch.4/4.4_LogTransformations.R)

   * [logearn_height](https://github.com/stan-dev/stan/blob/feature/ARM/src/models/ARM/Ch.4/logearn_height.stan): linear model with one predictor and natural log transformation  
```
lm (log_earnings ~ height)
```

   * [log10earn_height](https://github.com/stan-dev/stan/blob/feature/ARM/src/models/ARM/Ch.4/log10earn_height.stan): linear model with one predictor and log10 transformation  
```
lm (log10_earnings ~ height)
```

   * [logearn_height_male](https://github.com/stan-dev/stan/blob/feature/ARM/src/models/ARM/Ch.4/logearn_height_male.stan): linear model with two predictors and natural log transformation  
```
lm (log_earnings ~ height + male)
```

   * [logearn_interaction](https://github.com/stan-dev/stan/blob/feature/ARM/src/models/ARM/Ch.4/logearn_interaction.stan): linear model with two predictors and interaction and natural log transformation  
```
lm (log_earnings ~ height + male + height:male)
```

   * [logearn_interaction_z](https://github.com/stan-dev/stan/blob/feature/ARM/src/models/ARM/Ch.4/logearn_interaction_z.stan): linear model with two predictors and interaction and natural log transformation centered using z-score  
```
lm (log_earnings ~ z_height + male + z_height:male)
```

   * [logearn_logheight](https://github.com/stan-dev/stan/blob/feature/ARM/src/models/ARM/Ch.4/logearn_logheight.stan): linear model with two predictors and log log transformation  
```
lm (log_earnings ~ log_height + male)
```

  * [4.5 Other Transformations](https://github.com/stan-dev/stan/blob/feature/ARM/src/models/ARM/Ch.4/4.5_OtherTransformations.R)

   * [kidscore_momwork](https://github.com/stan-dev/stan/blob/feature/ARM/src/models/ARM/Ch.4/kidscore_momwork.stan): linear model with one factor   
```
lm (kid_score ~ as.factor(mom_work))
```

  * [4.6 Regression Models for Prediction](https://github.com/stan-dev/stan/blob/feature/ARM/src/models/ARM/Ch.4/4.6_RegressionModelsForPrediction.R)

   * [mesquite](https://github.com/stan-dev/stan/blob/feature/ARM/src/models/ARM/Ch.4/mesquite.stan): linear model with six predictors  
```
lm (weight~ diam1 + diam2 + canopy_height + total_height + density + group)
```

   * [mesquite_log](https://github.com/stan-dev/stan/blob/feature/ARM/src/models/ARM/Ch.4/mesquite_log.stan): linear model with six predictors and log transformation  
```
lm (log_weight~ log_diam1 + log_diam2 + log_canopy_height + log_total_height + log_density + group)
```

   * [mesquite_volume](https://github.com/stan-dev/stan/blob/feature/ARM/src/models/ARM/Ch.4/mesquite_volume.stan): linear model with one transformed predictor and log transformation  
```
lm (log_weight ~ log_canopy_volume)
```

   * [mesquite_vas](https://github.com/stan-dev/stan/blob/feature/ARM/src/models/ARM/Ch.4/mesquite_vas.stan): linear model with three predictors and three transformed predictors and log transformation           
```
lm (log_weight ~ log_canopy_volume + log_canopy_area + log_canopy_shape + log_total_height + log_density + group)
```

   * [mesquite_va](https://github.com/stan-dev/stan/blob/feature/ARM/src/models/ARM/Ch.4/mesquite_va.stan): linear model with one predictor and two transformed predictors and log transformation   
```
lm (log_weight ~ log_canopy_volume + log_canopy_area + group)
```

   * [mesquite_vash](https://github.com/stan-dev/stan/blob/feature/ARM/src/models/ARM/Ch.4/mesquite_vash.stan): linear model with two predictors and three transformed predictors and log transformation
```
lm (log_weight ~ log_canopy_volume + log_canopy_area + log_canopy_shape + log_total_height + group)
```

  * [4.7 Fitting a Series of Regressions - graph partially works](https://github.com/stan-dev/stan/blob/feature/ARM/src/models/ARM/Ch.4/4.7_FittingASeriesofRegressions.R)

   * [nes](https://github.com/stan-dev/stan/blob/feature/ARM/src/models/ARM/Ch.4/nes.stan): linear model with eight predictors    
```
lm (partyid7 ~ real_ideo + race_adj + age30_44 + age45_64 + age65up + educ1 + gender + income)
```

***

#### [Chapter 5 - Logistic Regression](https://github.com/stan-dev/stan/tree/feature/ARM/src/models/ARM/Ch.5)

  * [5.1 Logistic Regression with One Predictor](https://github.com/stan-dev/stan/blob/feature/ARM/src/models/ARM/Ch.5/5.1_LogisticRegressionWithOnePredictor.R)

   * [nes_logit](https://github.com/stan-dev/stan/blob/feature/ARM/src/models/ARM/Ch.5/nes_logit.stan): generalized linear model with  logit link function and one predictor  
```
glm (vote ~ income, family=binomial(link="logit"))
```

  * [5.2 Interpreting Logistic Regression Coeficients](https://github.com/stan-dev/stan/blob/feature/ARM/src/models/ARM/Ch.5/5.2_InterpretingLogisticRegressionCoef.R)

   * [nes_logit](https://github.com/stan-dev/stan/blob/feature/ARM/src/models/ARM/Ch.5/nes_logit.stan): generalized linear model with logit link function and one predictor  
```
glm (vote ~ income, family=binomial(link="logit"))
```

  * [5.4 Logistic Regression Wells in Bangladesh](https://github.com/stan-dev/stan/blob/feature/ARM/src/models/ARM/Ch.5/5.4_LogisticRegressionWellsinBangladesh.R)

   * [wells_dist](https://github.com/stan-dev/stan/blob/feature/ARM/src/models/ARM/Ch.5/wells_dist.stan): generalized linear model with logit link function and one predictor  
```
glm (switched ~ dist, family=binomial(link="logit"))
```

   * [wells_dist100](https://github.com/stan-dev/stan/blob/feature/ARM/src/models/ARM/Ch.5/wells_dist100.stan): generalized linear model with logit link function and one predictor  
```
glm (switched ~ dist100, family=binomial(link="logit"))
```

  * [5.5 Logistic Regression with Interactions](https://github.com/stan-dev/stan/blob/feature/ARM/src/models/ARM/Ch.5/5.5_LogisticRegressionWithInteractions.R)

   * [wells_interaction](https://github.com/stan-dev/stan/blob/feature/ARM/src/models/ARM/Ch.5/wells_interaction.stan): generalized linear model with logit link function and two predictors and interaction  
```
glm (switched ~ dist100 + arsenic + dist100:arsenic, family=binomial(link="logit"))
```

   * [wells_interaction_c](https://github.com/stan-dev/stan/blob/feature/ARM/src/models/ARM/Ch.5/wells_interaction_c.stan): generalized linear model with logit link function with two predictors and interaction centered using mean   
```
glm (switched ~ c_dist100 + c_arsenic + c_dist100:c_arsenic, family=binomial(link="logit"))
```

   * [wells_daae_c](https://github.com/stan-dev/stan/blob/feature/ARM/src/models/ARM/Ch.5/wells_daae_c.stan): generalized linear model with logit link function and four predictors and interaction centered using mean  
```
glm (switched ~ c_dist100 + c_arsenic + c_dist100:c_arsenic + assoc + educ4, family=binomial(link="logit"))
```

   * [wells_dae_c](https://github.com/stan-dev/stan/blob/feature/ARM/src/models/ARM/Ch.5/wells_dae_c.stan): generalized linear model with logit link function and three predictors and interaction centered using mean  
```
glm (switched ~ c_dist100 + c_arsenic + c_dist100:c_arsenic + educ4, family=binomial(link="logit"))
```

   * [wells_predicted](https://github.com/stan-dev/stan/blob/feature/ARM/src/models/ARM/Ch.5/wells_predicted.stan): generalized linear model with logit link function and three predictors and interaction centered using mean  
```
glm (switched ~ c_dist100 + c_arsenic + c_educ4 + c_dist100:c_arsenic + c_dist100:c_educ4 + c_arsenic:c_educ4, 
     family=binomial(link="logit"))
```

  * [5.6 Evaluating, Checking, & Comparing](https://github.com/stan-dev/stan/blob/feature/ARM/src/models/ARM/Ch.5/5.6_Evaluating%2CChecking%2C%26Comparing.R)

   * [wells_predicted](https://github.com/stan-dev/stan/blob/feature/ARM/src/models/ARM/Ch.5/wells_predicted.stan): generalized linear model with logit link function with three predictors and interaction centered using mean  
```
glm (switc ~ c_dist100 + c_arsenic + c_educ4 + c_dist100:c_arsenic + c_dist100:c_educ4 + c_arsenic:c_educ4, 
     family=binomial(link="logit"))
```

   * [wells_predicted_log](https://github.com/stan-dev/stan/blob/feature/ARM/src/models/ARM/Ch.5/wells_predicted_log.stan): generalized linear model with logit link function with three predictors and interaction with log transform and centered using mean   
```
glm (switched ~ c_dist100 + c_log_arsenic + c_educ4 + c_dist100:c_log_arsenic + c_dist100:c_educ4     
                + c_log_arsenic:c_educ4, 
     family=binomial(link="logit"))
```

   * [wells_predicted_log](https://github.com/stan-dev/stan/blob/feature/ARM/src/models/ARM/Ch.5/wells_predicted_log.stan): generalized linear model with logit link function with three predictors and interaction with log transform and centered using mean  
```
glm (switched ~ c_dist100 + c_log_arsenic + c_educ4 + c_dist100:c_log_arsenic + c_dist100:c_educ4     
                + c_log_arsenic:c_educ4, 
     family=binomial(link="logit"))
```

  * [5.7 Average Predictive Comparisons](https://github.com/stan-dev/stan/blob/feature/ARM/src/models/ARM/Ch.5/5.7_AveragePredictiveComparisons.R)

   * [wells_dae](https://github.com/stan-dev/stan/blob/feature/ARM/src/models/ARM/Ch.5/wells_dae.stan): generalized linear model with logit link function and three predictors  
```
glm (switched ~ dist100 + arsenic + educ4, family=binomial(link="logit"))
```

   * [wells_dae_inter](https://github.com/stan-dev/stan/blob/feature/ARM/src/models/ARM/Ch.5/wells_dae_inter.stan): generalized linear model with logit link function and three predictors with interaction  
```
glm (switched ~ dist100 + arsenic + educ4 + dist100:arsenic, family=binomial(link="logit"))
```

  * [5.8 Identifiability and Separating](https://github.com/stan-dev/stan/blob/feature/ARM/src/models/ARM/Ch.5/5.8_IdentifiabilityAndSeparation.R)

   * [separation](https://github.com/stan-dev/stan/blob/feature/ARM/src/models/ARM/Ch.5/separation.stan): generalized linear model with logit link function and one predictor  
```
glm (y ~ x, family=binomial(link="logit"))
```

***

#### [Chapter 6 - Generalized Linear Models](https://github.com/stan-dev/stan/tree/feature/ARM/src/models/ARM/Ch.6)

  * [6.2 Poisson, Exposure, & Overdispersion - MISSING DATA](https://github.com/stan-dev/stan/blob/feature/ARM/src/models/ARM/Ch.6/6.2_Poisson%2CExposure%26Overdispersion.R)

  * [6.4 Probit Regression](https://github.com/stan-dev/stan/blob/feature/ARM/src/models/ARM/Ch.6/6.4_ProbitRegression.R)

   * [wells_probit](https://github.com/stan-dev/stan/blob/feature/ARM/src/models/ARM/Ch.6/wells_probit.stan): generalized linear model with probit link function and one predictor  
```
glm (switc ~ dist100, family=binomial(link="probit"))
```

  * [6.5 Ordered & Unordered Categorical Regression - MISSING DATA](https://github.com/stan-dev/stan/blob/feature/ARM/src/models/ARM/Ch.6/6.5_Ordered%26UnorderedCategoricalRegression.R)

  * [6.7 More Complex GLM](https://github.com/stan-dev/stan/blob/feature/ARM/src/models/ARM/Ch.6/6.7_MoreComplexGLM.R)

   * [earnings1](https://github.com/stan-dev/stan/blob/feature/ARM/src/models/ARM/Ch.6/earnings1.stan): generalized linear model with logit link function and two predictors  
```
glm (earn_pos ~ height + male, family=binomial(link="logit"))
```

   * [earnings2](https://github.com/stan-dev/stan/blob/feature/ARM/src/models/ARM/Ch.6/earnings2.stan): linear model with two predictors and log transformation 
```
lm (log_earn ~ height + male, subset=earn>0)
```

  * [6.8 Constructive Choice Models](https://github.com/stan-dev/stan/blob/feature/ARM/src/models/ARM/Ch.6/6.8_ConstructiveChoiceModels.R)

   * [wells_logit](https://github.com/stan-dev/stan/blob/feature/ARM/src/models/ARM/Ch.6/wells_logit.stan): generalized linear model with logit link function and one predictor  
```
glm (switc ~ dist100, family=binomial(link="logit"))
```

***

#### [Chapter 7 - Simulation of Probability Models and Statistical Inferences](https://github.com/stan-dev/stan/tree/feature/ARM/src/models/ARM/Ch.7)

  * [7.1 Simulation of Probability Models](https://github.com/stan-dev/stan/blob/feature/ARM/src/models/ARM/Ch.7/7.1_SimulationOfProbabilityModels.R)

  * [7.2 Summarizing Linear Regression Using Simulation](https://github.com/stan-dev/stan/blob/feature/ARM/src/models/ARM/Ch.7/7.2_SummarizingLinearRegressionUsingSimulation.R)

   * [earnings_interactions](https://github.com/stan-dev/stan/blob/feature/ARM/src/models/ARM/Ch.7/earnings_interactions.stan): linear model with two predictors and interaction and log transformation   
```
lm (log_earn ~ height + male + height:male)
```

  * [7.3 Simulation for NonLinear Predictions](https://github.com/stan-dev/stan/blob/feature/ARM/src/models/ARM/Ch.7/7.3_SimulationForNonLinearPredictions.R)

   * [congress](https://github.com/stan-dev/stan/blob/feature/ARM/src/models/ARM/Ch.7/congress.stan): linear model with two predictors  
```
lm (vote_88 ~ vote_86 + incumbency_88)
```

  * [7.4 Predictive Simulation for GLM](https://github.com/stan-dev/stan/blob/feature/ARM/src/models/ARM/Ch.7/7.4_PredictiveSimulationForGLM.R)

   * [wells](https://github.com/stan-dev/stan/blob/feature/ARM/src/models/ARM/Ch.7/wells.stan): generalized linear model with logit link function and one predictor   
```
glm (switc ~ dist, family=binomial(link="logit"))
```

   * [earnings1](https://github.com/stan-dev/stan/blob/feature/ARM/src/models/ARM/Ch.7/earnings1.stan): generalized linear model with logit link function and two predictors   
```
glm (earn_pos ~ height + male, family=binomial(link="logit"))
```
 
   * [earnings2](https://github.com/stan-dev/stan/blob/feature/ARM/src/models/ARM/Ch.7/earnings2.stan): linear model with two predictors and log transformation 
```
lm (log_earn ~ height + male, subset=earn>0)
```

***

#### [Chapter 8 - Simulation for Checking Statistical Procedures and Model Fits](https://github.com/stan-dev/stan/tree/feature/ARM/src/models/ARM/Ch.8)

  * [8.1 Fake Data Simulation](https://github.com/stan-dev/stan/blob/feature/ARM/src/models/ARM/Ch.8/8.1_FakeDataSimulation.R)

   * [y_x](https://github.com/stan-dev/stan/blob/feature/ARM/src/models/ARM/Ch.8/y_x.stan): linear model with one predictor
```
lm (y ~ x)
```

  * [8.2 Fake Data Simulation to Understand Residual Plots](https://github.com/stan-dev/stan/blob/feature/ARM/src/models/ARM/Ch.8/8.2_FakeDataSimulationToUnderstandResidualPlots.R)

   * [grades](https://github.com/stan-dev/stan/blob/feature/ARM/src/models/ARM/Ch.8/grades.stan): linear model with one predictor
```
lm (final ~ midterm)
```

  * [8.3 Simulating from the Fitted Model](https://github.com/stan-dev/stan/blob/feature/ARM/src/models/ARM/Ch.8/8.3_SimulatingFromTheFittedModel.R)

   * [lightspeed](https://github.com/stan-dev/stan/blob/feature/ARM/src/models/ARM/Ch.8/lightspeed.stan): linear model with no predictors
```
lm (y ~ 1)
```

   * [roaches](https://github.com/stan-dev/stan/blob/feature/ARM/src/models/ARM/Ch.8/roaches.stan): poisson regression model with exposure and three predictors
```
glm (y ~ roach1 + treatment + senior, family=poisson, offset=log(exposure2))
```

   * [roaches_overdispersion](https://github.com/stan-dev/stan/blob/feature/ARM/src/models/ARM/Ch.8/roaches_overdispersion.stan): poisson overdispersion regression model with exposure and three predictors
```
glm(y ~ roach1 + treatment + senior, family=quasipoisson, offset=log(exposure2))
```

  * [8.4 Predictive Simulation to Check Fit of Time Series Model](https://github.com/stan-dev/stan/blob/feature/ARM/src/models/ARM/Ch.8/8.4_PredictiveSimulationToCheckFitOfTimeSeriesModels.R)

   * [unemployment](https://github.com/stan-dev/stan/blob/feature/ARM/src/models/ARM/Ch.8/unemployment.stan): linear model with one predictor  
```
lm (y ~ y_lag)
```

***

#### [Chapter 9 - Casual Inference Using Regression on the Treatment Variable](https://github.com/stan-dev/stan/tree/feature/ARM/src/models/ARM/Ch.9)

  * [9.3 Randomized Experiments - convert regression.2tables](https://github.com/stan-dev/stan/blob/feature/ARM/src/models/ARM/Ch.9/9.3_RandomizedExperiments.R)

   * [electric_tr](https://github.com/stan-dev/stan/blob/feature/ARM/src/models/ARM/Ch.9/electric_tr.stan): linear model with one predictor
```
lm (post_test ~ treatment)
```

   * [electric_trpre](https://github.com/stan-dev/stan/blob/feature/ARM/src/models/ARM/Ch.9/electric_trpre.stan): linear model with two predictors  
```
lm (post_test ~ pre_test + treatment)
```

  * [9.4 Treatment Interactions & Post Stratification](https://github.com/stan-dev/stan/blob/feature/ARM/src/models/ARM/Ch.9/9.4_TreatmentInteractions%26Poststratification.R)

   * [electric_tr](https://github.com/stan-dev/stan/blob/feature/ARM/src/models/ARM/Ch.9/electric_tr.stan): linear model with one predictor    
```
lm (post_test ~ treatment)
```

   * [electric_trpre](https://github.com/stan-dev/stan/blob/feature/ARM/src/models/ARM/Ch.9/electric_trpre.stan): linear model with two predictors  
```
lm (post_test ~ treatment + pre_test)
```

   * [electric_inter](https://github.com/stan-dev/stan/blob/feature/ARM/src/models/ARM/Ch.9/electric_inter.stan): linear model with two predictors and interaction  
```
lm (post_test ~ pre_test + treatment + pre_test:treatment)
```

  * [9.5 Observational Studies - havent started](https://github.com/stan-dev/stan/blob/feature/ARM/src/models/ARM/Ch.9/9.5_ObservationalStudies.R)

   * [electric_supp](https://github.com/stan-dev/stan/blob/feature/ARM/src/models/ARM/Ch.9/electric_supp.stan): linear model with two predictors and interaction  
```
lm (post_test ~ supp + pre_test)
```

***

#### [Chapter 10 - Casual Inference Using More Advanced Models](https://github.com/stan-dev/stan/tree/feature/ARM/src/models/ARM/Ch.10)

  * [10.3 Matching - MISSING DATA](https://github.com/stan-dev/stan/blob/feature/ARM/src/models/ARM/Ch.10/10.3_Matching.R)

  * [10.4 Lack of Overlap when Treat Assignments is Unknown](https://github.com/stan-dev/stan/blob/feature/ARM/src/models/ARM/Ch.10/10.4_LackOfOverlapWhenTreat.AssignmentIsUnknown.R)

   * [ideo_two_pred](https://github.com/stan-dev/stan/blob/feature/ARM/src/models/ARM/Ch.10/ideo_two_pred.stan): linear model with two predictors  
```
lm (score1 ~ party + x, subset=overlap)
```

   * [ideo_two_pred](https://github.com/stan-dev/stan/blob/feature/ARM/src/models/ARM/Ch.10/ideo_two_pred.stan): linear model with two predictors  
```
lm (score1 ~ party + x, subset=incs)
```

   * [ideo_reparam](https://github.com/stan-dev/stan/blob/feature/ARM/src/models/ARM/Ch.10/ideo_reparam.stan): linear model with two predictors and reparamaterization  
```
lm (score1 ~ party + I(z*(party==0)) + I(z*(party==1)), subset=incs)
```
 
   * [ideo_interactions](https://github.com/stan-dev/stan/blob/feature/ARM/src/models/ARM/Ch.10/ideo_interactions.stan): linear model with two predictors and interaction  
```
lm (score1 ~ party + x + party:x, subset=incs)
```

  * [10.5 Casual Effects Using IV](https://github.com/stan-dev/stan/blob/feature/ARM/src/models/ARM/Ch.10/10.5_CasualEffectsUsingIV.R)

   * [sesame_one_pred_a](https://github.com/stan-dev/stan/blob/feature/ARM/src/models/ARM/Ch.10/sesame_one_pred_a.stan): linear model with one predictor  
```
lm (watched ~ encouraged)
```

   * [sesame_one_pred_b](https://github.com/stan-dev/stan/blob/feature/ARM/src/models/ARM/Ch.10/sesame_one_pred_b.stan): linear model with one predictor  
```
lm (y ~ encouraged)
```

  * [10.6 IV in a Regression Framework - misisng stuff at bottom](https://github.com/stan-dev/stan/blob/feature/ARM/src/models/ARM/Ch.10/10.6_IVinaRegressionFramework.R)

   * [sesame_one_pred_a](https://github.com/stan-dev/stan/blob/feature/ARM/src/models/ARM/Ch.10/sesame_one_pred_a.stan): linear model with one predictor  
```
lm (watched ~ encouraged)
```

   * [sesame_one_pred_2b](https://github.com/stan-dev/stan/blob/feature/ARM/src/models/ARM/Ch.10/sesame_one_pred_2b.stan): linear model with one predictor  
```
lm (y ~ watched_hat)
```

   * [sesame_multi_preds_3a](https://github.com/stan-dev/stan/blob/feature/ARM/src/models/ARM/Ch.10/sesame_multi_preds_3a.stan): linear model with three predictors and one factor  
```
lm (watched ~ encouraged + pretest + as.factor(site) + setting)
```

   * [sesame_multi_preds_3b](https://github.com/stan-dev/stan/blob/feature/ARM/src/models/ARM/Ch.10/sesame_multi_preds_3b.stan): linear model with three predictors and one factor  
```
lm (y ~ watched_hat + pretest + as.factor(site) + setting)
```

***

#### [Chapter 11 - Multilevel Structures](https://github.com/stan-dev/stan/tree/feature/ARM/src/models/ARM/Ch.11)
 
  * [11.3 Repeated Measurements, Time-Series, Cross Sections & Others - missing data](https://github.com/stan-dev/stan/blob/feature/ARM/src/models/ARM/Ch.11/11.3_RepeatedMeasurements%2CTimeSeries%20CrossSections%26Others.R)

***

#### [Chapter 12 - Multilevel Linear Models: the Basics](https://github.com/stan-dev/stan/tree/feature/ARM/src/models/ARM/Ch.12)

  * [12.2 Partial Pooling with No Predictors](https://github.com/stan-dev/stan/blob/feature/ARM/src/models/ARM/Ch.12/12.2_PartialPoolingWithNoPredictors.R)

    * [radon_intercept](https://github.com/stan-dev/stan/blob/feature/ARM/src/models/ARM/Ch.12/radon_intercept.stan): multi-level linear model with varying intercept        
```
lmer (y ~ 1 + (1 | county))
```

    * [radon_intercept_chr](https://github.com/stan-dev/stan/blob/feature/ARM/src/models/ARM/Ch.12/radon_intercept_chr.stan): multi-level linear model with varying intercept using the Choo-Hoffman Parametrization        
```
lmer (y ~ 1 + (1 | county))
```

  * [12.3 Partial Pooling with Predictors](https://github.com/stan-dev/stan/blob/feature/ARM/src/models/ARM/Ch.12/12.3_PartialPoolingWithPredictors.R) 

    * [radon_complete_pool](https://github.com/stan-dev/stan/blob/feature/ARM/src/models/ARM/Ch.12/radon_complete_pool.stan): multi-level linear model with complete pooling          
```
lm (y ~ x)
```

    * [radon_no_pool](https://github.com/stan-dev/stan/blob/feature/ARM/src/models/ARM/Ch.12/radon_no_pool.stan): multi-level linear model with no pooling        
```
lmer (y ~ x + (1 | county))
```

    * [radon_no_pool_chr](https://github.com/stan-dev/stan/blob/feature/ARM/src/models/ARM/Ch.12/radon_no_pool_chr.stan): multi-level linear model with no pooling using the Choo-Hoffman Parametrization       
```
lmer (y ~ x + (1 | county))
```

  * [12.4 Fitting MLM in R](https://github.com/stan-dev/stan/blob/feature/ARM/src/models/ARM/Ch.12/12.4_FittingMLMinR.R)

    * [radon_intercept](https://github.com/stan-dev/stan/blob/feature/ARM/src/models/ARM/Ch.12/radon_intercept.stan): multi-level linear model with varying intercept         
```
lmer (y ~ 1 + (1 | county))
```

    * [radon_no_pool](https://github.com/stan-dev/stan/blob/feature/ARM/src/models/ARM/Ch.12/radon_no_pool.stan): multi-level linear model with no pooling          
```
lmer (y ~ x + (1 | county))
```

  * [12.6 Group-Level Predictors](https://github.com/stan-dev/stan/blob/feature/ARM/src/models/ARM/Ch.12/12.6_Group-LevelPredictors.R)

    * [radon_group](https://github.com/stan-dev/stan/blob/feature/ARM/src/models/ARM/Ch.12/radon_group.stan): multi-level linear model with group level predictor and individual level predictors              
```
lmer (y ~ x + u + (1 | county))
```

    * [radon_group_chr](https://github.com/stan-dev/stan/blob/feature/ARM/src/models/ARM/Ch.12/radon_group_chr.stan): multi-level linear model with group level predictor and individual level predictors using the Choo-Hoffman Parametrization             
```
lmer (y ~ x + u + (1 | county))
```

    * [radon_no_pool](https://github.com/stan-dev/stan/blob/feature/ARM/src/models/ARM/Ch.12/radon_no_pool.stan): multi-level linear model with no pooling                
```
lmer (y ~ x + (1 | county))
```

  * [12.8 Prediction](https://github.com/stan-dev/stan/blob/feature/ARM/src/models/ARM/Ch.12/12.8_Prediction.R)

    * [radon_group](https://github.com/stan-dev/stan/blob/feature/ARM/src/models/ARM/Ch.12/radon_group.stan): multi-level linear model with group level predictor and individual level predictors         
```
lmer (y ~ x + u + (1 | county))
```

***

#### [Chapter 13 - Multilevel Linear Models: Varying Slopes, Non-Nested Models, and Other Complexities](https://github.com/stan-dev/stan/tree/feature/ARM/src/models/ARM/Ch.13)

  * [13.1 Varying Intercepts & Slopes](https://github.com/stan-dev/stan/blob/feature/ARM/src/models/ARM/Ch.13/13.1_VaryingIntercepts%26Slopes.R)

    * [radon_vary_si](https://github.com/stan-dev/stan/blob/feature/ARM/src/models/ARM/Ch.13/radon_vary_si.stan): multi-level linear model with group level predictors         
```
lmer (y ~ x (1 + x | county))
```

    * [radon_vary_si_chr](https://github.com/stan-dev/stan/blob/feature/ARM/src/models/ARM/Ch.13/radon_vary_si_chr.stan): multi-level linear model with group level predictors using the Choo-Hoffman Parametrization        
```
lmer (y ~ x (1 + x | county))
```

   * [y_x](https://github.com/stan-dev/stan/blob/feature/ARM/src/models/ARM/Ch.13/y_x.stan): linear model with one predictor       
```
lm (y ~ x)
```

    * [radon_inter_vary](https://github.com/stan-dev/stan/blob/feature/ARM/src/models/ARM/Ch.13/radon_inter_vary.stan): multi-level linear model with group level predictors         
```
lmer (y ~ x + u.full + x:u.full + (1 + x | county))
```

    * [radon_inter_vary_chr](https://github.com/stan-dev/stan/blob/feature/ARM/src/models/ARM/Ch.13/radon_inter_vary_chr.stan): multi-level linear model with group level predictors using the Choo-Hoffman Parametrization        
```
lmer (y ~ x + u.full + x:u.full + (1 + x | county))
```

  * [13.4 Understanding Correlations Between Intercepts & Slopes](https://github.com/stan-dev/stan/blob/feature/ARM/src/models/ARM/Ch.13/13.4_UnderstandingCorrelationsBetweenIntercepts%26Slopes.R)
 
   * [earnings_vary_si](https://github.com/stan-dev/stan/blob/feature/ARM/src/models/ARM/Ch.13/earnings_vary_si.stan): multi-level linear model with group level predictors         
```
lmer (y ~ x (1 + x | ethn))
```
 
   * [earnings_vary_si_chr](https://github.com/stan-dev/stan/blob/feature/ARM/src/models/ARM/Ch.13/earnings_vary_si_chr.stan): multi-level linear model with group level predictors using the Choo-Hoffman Parametrization        
```
lmer (y ~ x (1 + x | ethn))
```

  * [13.5 Non-Nested Models](https://github.com/stan-dev/stan/blob/feature/ARM/src/models/ARM/Ch.13/13.5_Non-NestedModels.R)
 
   * [pilots](https://github.com/stan-dev/stan/blob/feature/ARM/src/models/ARM/Ch.13/pilots.stan): non-nested multi-level linear model with group level predictors         
```
lmer (y ~ 1 + (1 | group.id) (1 | scenario.id))
```
  
   * [pilots_chr](https://github.com/stan-dev/stan/blob/feature/ARM/src/models/ARM/Ch.13/pilots_chr.stan): non-nested multi-level linear model with group level predictors using the Choo-Hoffman Parametrization        
```
lmer (y ~ 1 + (1 | group.id) (1 | scenario.id))
```

   * [earnings_latin_square](https://github.com/stan-dev/stan/blob/feature/ARM/src/models/ARM/Ch.13/earnings_latin_square.stan): non-nested multi-level linear model with group level predictors
```
lmer (y ~ x.centered + (1 + x.centered | eth) + (1 + x.centered | age) + (1 + x.centered | eth:age))
```

   * [earnings_latin_square_chr](https://github.com/stan-dev/stan/blob/feature/ARM/src/models/ARM/Ch.13/earnings_latin_square_chr.stan): non-nested multi-level linear model with group level predictors using the Choo-Hoffman Parametrization
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

    * [radon_chr](https://github.com/stan-dev/stan/blob/feature/ARM/src/models/ARM/Ch.19/radon_chr.stan): multi-level liner model with varying intercept using the Choo-Hoffman Parametrization
```
lmer (y ~ 1 + (1 | county))
```

    * [radon_redundant](https://github.com/stan-dev/stan/blob/feature/ARM/src/models/ARM/Ch.19/radon_redundant.stan): multi-level liner model with varying intercept and redundant parameterization
```
lmer (y ~ 1 + (1 | county))
```

    * [radon_redundant_chr](https://github.com/stan-dev/stan/blob/feature/ARM/src/models/ARM/Ch.19/radon_redundant_chr.stan): multi-level liner model with varying intercept and redundant parameterization and the Choo-Hoffman Parametrization
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

    * [hiv_chr](https://github.com/stan-dev/stan/blob/feature/ARM/src/models/ARM/Ch.20/hiv_chr.stan): multi-level linear model with varying slope and intercept using the Choo-Hoffman Parametrization
```
lmer (y ~ time + (1 + time | person)
```

    * [hiv_inter](https://github.com/stan-dev/stan/blob/feature/ARM/src/models/ARM/Ch.20/hiv_inter.stan): multi-level linear model with interaction and varying slope and intercept
```
lmer (y ~ time:treatment + (1 + time | person)
```

    * [hiv_inter_chr](https://github.com/stan-dev/stan/blob/feature/ARM/src/models/ARM/Ch.20/hiv_inter_chr.stan): multi-level linear model with interaction and varying slope and intercept using the Choo-Hoffman Parametrization
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

    * [radon_vary_intercept_floor_chr](https://github.com/stan-dev/stan/blob/feature/ARM/src/models/ARM/Ch.21/radon_vary_intercept_floor_chr.stan): multi-level linear model with varying intercept using the Choo-Hoffman Parametrization
```
lmer (y ~ u + x + (1 | county))
```

    * [radon_vary_intercept_floor2](https://github.com/stan-dev/stan/blob/feature/ARM/src/models/ARM/Ch.21/radon_vary_intercept_floor2.stan): multi-level linear model with varying intercept
```
lmer (y ~ u + x + x_mean + (1 | county))
```

    * [radon_vary_intercept_floor2_chr](https://github.com/stan-dev/stan/blob/feature/ARM/src/models/ARM/Ch.21/radon_vary_intercept_floor2_chr.stan): multi-level linear model with varying intercept using the Choo-Hoffman Parametrization
```
lmer (y ~ u + x + x_mean + (1 | county))
```

    * [radon_vary_intercept_nofloor](https://github.com/stan-dev/stan/blob/feature/ARM/src/models/ARM/Ch.21/radon_vary_intercept_nofloor.stan): multi-level linear model with varying intercept
```
lmer (y ~ u + (1 | county))
```

    * [radon_vary_intercept_nofloor_chr](https://github.com/stan-dev/stan/blob/feature/ARM/src/models/ARM/Ch.21/radon_vary_intercept_nofloor_chr.stan): multi-level linear model with varying intercept using the Choo-Hoffman Parametrization
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

    * [anova_radon_nopred_chr](https://github.com/stan-dev/stan/blob/feature/ARM/src/models/ARM/Ch.22/anova_radon_nopred_chr.stan): multi-level linear model with varying intercept and set up for ANOVA using the Choo-Hoffman Parametrization
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

    * [electric_1a_chr](https://github.com/stan-dev/stan/blob/feature/ARM/src/models/ARM/Ch.23/electric_1a_chr.stan): multi-level linear model with varying intercept and slope using the Choo-Hoffman Parametrization
```
lmer (y ~ 1 + (1 | pair) + (treatment | grade))
```

    * [electric_1b](https://github.com/stan-dev/stan/blob/feature/ARM/src/models/ARM/Ch.23/electric_1b.stan): multi-level linear model with varying intercept and slope
```
lmer (y ~ treatment + pre_test + (1 | pair))
```

    * [electric_1b_chr](https://github.com/stan-dev/stan/blob/feature/ARM/src/models/ARM/Ch.23/electric_1b_chr.stan): multi-level linear model with varying intercept and slope using the Choo-Hoffman Parametrization
```
lmer (y ~ treatment + pre_test + (1 | pair))
```

    * [electric_1c](https://github.com/stan-dev/stan/blob/feature/ARM/src/models/ARM/Ch.23/electric_1c.stan): multi-level linear model with group level factors
```
lmer (y ~ 1 + (1 | pair) + (treatment | grade) + (pre_test | grade))
```

    * [electric_1c_chr](https://github.com/stan-dev/stan/blob/feature/ARM/src/models/ARM/Ch.23/electric_1c_chr.stan): multi-level linear model with group level factors using the Choo-Hoffman Parametrization
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

    * [electric_chr](https://github.com/stan-dev/stan/blob/feature/ARM/src/models/ARM/Ch.23/electric_chr.stan): multi-level linear model with varying intercept using the Choo-Hoffman Parametrization
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
 