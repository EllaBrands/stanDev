Each chapter has a README file on github that you can view by clicking on the chapter link and scrolling down (past the files). THe README file contains information about the data contained in the chapter (particularly, what each variable represents) and sorts the models in the chapter by type.

If there is a * next to a model name, then the model DOES NOT currently work with [RStanARM](https://github.com/stan-dev/rstanarm). Furthermore, multilevel models are not currently supported by RStanARM, but all other models below (without a *) are supported.

* [Chapter 2](https://github.com/stan-dev/stan/wiki/ARM-Models-Sorted-by-Chapter#chapter-2)
* [Chapter 3](https://github.com/stan-dev/stan/wiki/ARM-Models-Sorted-by-Chapter#chapter-3)
* [Chapter 4](https://github.com/stan-dev/stan/wiki/ARM-Models-Sorted-by-Chapter#chapter-4)
* [Chapter 5](https://github.com/stan-dev/stan/wiki/ARM-Models-Sorted-by-Chapter#chapter-5)
* [Chapter 6](https://github.com/stan-dev/stan/wiki/ARM-Models-Sorted-by-Chapter#chapter-6)
* [Chapter 7](https://github.com/stan-dev/stan/wiki/ARM-Models-Sorted-by-Chapter#chapter-7)
* [Chapter 8](https://github.com/stan-dev/stan/wiki/ARM-Models-Sorted-by-Chapter#chapter-8)
* [Chapter 9](https://github.com/stan-dev/stan/wiki/ARM-Models-Sorted-by-Chapter#chapter-9)
* [Chapter 10](https://github.com/stan-dev/stan/wiki/ARM-Models-Sorted-by-Chapter#chapter-10)
* [Chapter 11](https://github.com/stan-dev/stan/wiki/ARM-Models-Sorted-by-Chapter#chapter-11)
* [Chapter 12](https://github.com/stan-dev/stan/wiki/ARM-Models-Sorted-by-Chapter#chapter-12)
* [Chapter 13](https://github.com/stan-dev/stan/wiki/ARM-Models-Sorted-by-Chapter#chapter-13)
* [Chapter 14](https://github.com/stan-dev/stan/wiki/ARM-Models-Sorted-by-Chapter#chapter-14)
* [Chapter 15](https://github.com/stan-dev/stan/wiki/ARM-Models-Sorted-by-Chapter#chapter-15)
* [Chapter 16](https://github.com/stan-dev/stan/wiki/ARM-Models-Sorted-by-Chapter#chapter-16)
* [Chapter 17](https://github.com/stan-dev/stan/wiki/ARM-Models-Sorted-by-Chapter#chapter-17)
* [Chapter 18](https://github.com/stan-dev/stan/wiki/ARM-Models-Sorted-by-Chapter#chapter-18)
* [Chapter 19](https://github.com/stan-dev/stan/wiki/ARM-Models-Sorted-by-Chapter#chapter-19)
* [Chapter 20](https://github.com/stan-dev/stan/wiki/ARM-Models-Sorted-by-Chapter#chapter-20)
* [Chapter 21](https://github.com/stan-dev/stan/wiki/ARM-Models-Sorted-by-Chapter#chapter-21)
* [Chapter 22](https://github.com/stan-dev/stan/wiki/ARM-Models-Sorted-by-Chapter#chapter-22)
* [Chapter 23](https://github.com/stan-dev/stan/wiki/ARM-Models-Sorted-by-Chapter#chapter-23)
* [Chapter 24](https://github.com/stan-dev/stan/wiki/ARM-Models-Sorted-by-Chapter#chapter-24)
* [Chapter 25](https://github.com/stan-dev/stan/wiki/ARM-Models-Sorted-by-Chapter#chapter-25)


***

***

#### [Chapter 2](https://github.com/stan-dev/stan/tree/feature/ARM/src/models/ARM/Ch.2)
 * [2.3 CI](https://github.com/stan-dev/stan/blob/feature/ARM/src/models/ARM/Ch.2/2.3_CI.R)
 * [2.4 Hypothesis Testing - MISSING DATA](https://github.com/stan-dev/stan/blob/feature/ARM/src/models/ARM/Ch.2/2.4_HypothesisTesting.R)
 * [2.6 55,000 Residents - MISSING DATA](https://github.com/stan-dev/stan/blob/feature/ARM/src/models/ARM/Ch.2/2.6_55%2C000Residents.R)

***

#### [Chapter 3](https://github.com/stan-dev/stan/tree/feature/ARM/src/models/ARM/Ch.3)
 * [3.1 One Predictor](https://github.com/stan-dev/stan/blob/feature/ARM/src/models/ARM/Ch.3/3.1_OnePredictor.R)
   * [kid_iq](https://github.com/stan-dev/stan/blob/feature/ARM/src/models/ARM/Ch.3/kidscore_momhs.stan): linear model with one predictor  
    ``kid_score ~ mom_hs``
   * [kid_iq](https://github.com/stan-dev/stan/blob/feature/ARM/src/models/ARM/Ch.3/kidscore_momiq.stan): linear model with one predictor  
    ``kid_score ~ mom_iq``
 * [3.2 Multiple Predictors](https://github.com/stan-dev/stan/blob/feature/ARM/src/models/ARM/Ch.3/3.2_MultiplePredictors.R)
   * [kid_iq](https://github.com/stan-dev/stan/blob/feature/ARM/src/models/ARM/Ch.3/kidiq_multi_preds.stan): linear model with two predictors  
    ``kid_score ~ mom_hs + mom_iq``
 * [3.3 Interactions](https://github.com/stan-dev/stan/blob/feature/ARM/src/models/ARM/Ch.3/3.3_Interactions.R)
   * [kid_iq](https://github.com/stan-dev/stan/blob/feature/ARM/src/models/ARM/Ch.3/kidiq_interaction.stan): linear model with two predictors and interaction  
    ``kid_score ~ mom_hs + mom_iq + mom_hs:mom_iq``
 * [3.4 Stat Inference](https://github.com/stan-dev/stan/blob/feature/ARM/src/models/ARM/Ch.3/3.4_StatInference.R)
   * [kid_iq](https://github.com/stan-dev/stan/blob/feature/ARM/src/models/ARM/Ch.3/kidiq_multi_preds.stan): linear model with two predictors  
    ``kid_score ~ mom_hs + mom_iq``
 * [3.5 Graph Displays](https://github.com/stan-dev/stan/blob/feature/ARM/src/models/ARM/Ch.3/3.5_GraphDisplays.R)
   * [kid_iq](https://github.com/stan-dev/stan/blob/feature/ARM/src/models/ARM/Ch.3/kidscore_momiq.stan): linear model with one predictor  
    ``kid_score ~ mom_iq``
   * [kid_iq](https://github.com/stan-dev/stan/blob/feature/ARM/src/models/ARM/Ch.3/kidiq_multi_preds.stan): linear model with two predictors  
    ``kid_score ~ mom_hs + mom_iq``
   * [kid_iq](https://github.com/stan-dev/stan/blob/feature/ARM/src/models/ARM/Ch.3/kidiq_interaction.stan): linear model with two predictors and interaction  
    ``kid_score ~ mom_hs + mom_iq + mom_hs:mom_iq``
 * [3.6 Diagnostics](https://github.com/stan-dev/stan/blob/feature/ARM/src/models/ARM/Ch.3/3.6_Diagnostics.R)
   * [kid_iq](https://github.com/stan-dev/stan/blob/feature/ARM/src/models/ARM/Ch.3/kidscore_momiq.stan): linear model with one predictor  
    ``kid_score ~ mom_iq``
 * [3.7 Prediction](https://github.com/stan-dev/stan/blob/feature/ARM/src/models/ARM/Ch.3/3.7_Prediction.R)
   * [kid_iq](https://github.com/stan-dev/stan/blob/feature/ARM/src/models/ARM/Ch.3/kidiq_multi_preds.stan): linear model with two predictors  
    ``kid_score ~ mom_hs + mom_iq``
   * [kid_iq](https://github.com/stan-dev/stan/blob/feature/ARM/src/models/ARM/Ch.3/kidiq_validation.stan): linear model with two predictors  
    ``ppvt ~ hs + afqt``

***

#### [Chapter 4](https://github.com/stan-dev/stan/tree/feature/ARM/src/models/ARM/Ch.4)
  * [4.1 Linear Transformations](https://github.com/stan-dev/stan/blob/feature/ARM/src/models/ARM/Ch.4/4.1_LinearTransformations.R)
   * [earnings](https://github.com/stan-dev/stan/blob/feature/ARM/src/models/ARM/Ch.4/earn_height.stan): linear model with one predictor  
    ``earnings ~ height``
  * [4.2 Centering & Standardizing](https://github.com/stan-dev/stan/blob/feature/ARM/src/models/ARM/Ch.4/4.2_Centering%26Standardizing.R)
   * [kid_iq](https://github.com/stan-dev/stan/blob/feature/ARM/src/models/ARM/Ch.4/kidiq_interaction.stan): linear model with two predictors and interaction  
    ``kid_score ~ mom_hs + mom_iq + mom_hs:mom_iq``
   * [kid_iq](https://github.com/stan-dev/stan/blob/feature/ARM/src/models/ARM/Ch.4/kidiq_interaction_c.stan): linear model with two predictors and interaction centered using mean  
    ``kid_score ~ c_mom_hs + c_mom_iq + c_mom_hs:c_mom_iq``
   * [kid_iq](https://github.com/stan-dev/stan/blob/feature/ARM/src/models/ARM/Ch.4/kidiq_interaction_c2.stan): linear model with two predictors and interaction centered using conventional points  
    ``kid_score ~ c2_mom_hs + c2_mom_iq + c2_mom_hs:c2_mom_iq``
   * [kid_iq](https://github.com/stan-dev/stan/blob/feature/ARM/src/models/ARM/Ch.4/kidiq_interaction_z.stan): linear model with two predictors and interaction centered using z-score  
    ``kid_score ~ z_mom_hs + z_mom_iq + z_mom_hs:z_mom_iq``
  * [4.4 Log Transformations](https://github.com/stan-dev/stan/blob/feature/ARM/src/models/ARM/Ch.4/4.4_LogTransformations.R)
   * [earnings](https://github.com/stan-dev/stan/blob/feature/ARM/src/models/ARM/Ch.4/logearn_height.stan): linear model with one predictor and natural log transformation  
    ``log_earnings ~ height``
   * [earnings](https://github.com/stan-dev/stan/blob/feature/ARM/src/models/ARM/Ch.4/log10earn_height.stan): linear model with one predictor and log10 transformation  
    ``log10_earnings ~ height``
   * [earnings](https://github.com/stan-dev/stan/blob/feature/ARM/src/models/ARM/Ch.4/logearn_height_male.stan): linear model with two predictors and natural log transformation  
    ``log_earnings ~ height + male``
   * [earnings](https://github.com/stan-dev/stan/blob/feature/ARM/src/models/ARM/Ch.4/logearn_interaction.stan): linear model with two predictors and interaction and natural log transformation  
    ``log_earnings ~ height + male + height:male``
   * [earnings](https://github.com/stan-dev/stan/blob/feature/ARM/src/models/ARM/Ch.4/logearn_interaction_z.stan): linear model with two predictors and interaction and natural log transformation centered using z-score  
    ``log_earnings ~ z_height + male + z_height:male``
   * [earnings](https://github.com/stan-dev/stan/blob/feature/ARM/src/models/ARM/Ch.4/logearn_logheight.stan): linear model with two predictors and log log transformation  
    ``log_earnings ~ log_height + male``
  * [4.5 Other Transformations](https://github.com/stan-dev/stan/blob/feature/ARM/src/models/ARM/Ch.4/4.5_OtherTransformations.R)
   * [kid_iq](https://github.com/stan-dev/stan/blob/feature/ARM/src/models/ARM/Ch.4/kidscore_momwork.stan): linear model with one factor   
    ``kid_score ~ as.factor(mom_work)``
  * [4.6 Regression Models for Prediction](https://github.com/stan-dev/stan/blob/feature/ARM/src/models/ARM/Ch.4/4.6_RegressionModelsForPrediction.R)
   * [mesquite](https://github.com/stan-dev/stan/blob/feature/ARM/src/models/ARM/Ch.4/mesquite.stan): linear model with six predictors  
    ``weight~ diam1 + diam2 + canopy_height + total_height + density + group``
   * [mesquite](https://github.com/stan-dev/stan/blob/feature/ARM/src/models/ARM/Ch.4/mesquite_log.stan): linear model with six predictors and log transformation  
    ``log_weight~ log_diam1 + log_diam2 + log_canopy_height + log_total_height + log_density + group``
   * [mesquite](https://github.com/stan-dev/stan/blob/feature/ARM/src/models/ARM/Ch.4/mesquite_volume.stan): linear model with one transformed predictor and log transformation  
    ``log_weight ~ log_canopy_volume``
   * [mesquite](https://github.com/stan-dev/stan/blob/feature/ARM/src/models/ARM/Ch.4/mesquite_vas.stan): linear model with three predictors and three transformed predictors and log transformation           
    ``log_weight ~ log_canopy_volume + log_canopy_area + log_canopy_shape + log_total_height + log_density + group``
   * [mesquite](https://github.com/stan-dev/stan/blob/feature/ARM/src/models/ARM/Ch.4/mesquite_va.stan): linear model with one predictor and two transformed predictors and log transformation   
    ``log_weight ~ log_canopy_volume + log_canopy_area + group``
   * [mesquite](https://github.com/stan-dev/stan/blob/feature/ARM/src/models/ARM/Ch.4/mesquite_vash.stan): linear model with two predictors and three transformed predictors and log transformation  
    `` log_weight ~ log_canopy_volume + log_canopy_area + log_canopy_shape + log_total_height + group`` 
  * [4.7 Fitting a Series of Regressions - graph partially works](https://github.com/stan-dev/stan/blob/feature/ARM/src/models/ARM/Ch.4/4.7_FittingASeriesofRegressions.R)
   * [nes](https://github.com/stan-dev/stan/blob/feature/ARM/src/models/ARM/Ch.4/nes.stan): linear model with eight predictors    
    ``partyid7 ~ real_ideo + race_adj + age30_44 + age45_64 + age65up + educ1 + gender + income``

***

#### [Chapter 5](https://github.com/stan-dev/stan/tree/feature/ARM/src/models/ARM/Ch.5)
  * [5.1 Logistic Regression with One Predictor](https://github.com/stan-dev/stan/blob/feature/ARM/src/models/ARM/Ch.5/5.1_LogisticRegressionWithOnePredictor.R)
   * [nes](https://github.com/stan-dev/stan/blob/feature/ARM/src/models/ARM/Ch.5/nes.stan): generalized linear model with  logit link function and one predictor  
     ``vote ~ income``
  * [5.2 Interpreting Logistic Regression Coeficients](https://github.com/stan-dev/stan/blob/feature/ARM/src/models/ARM/Ch.5/5.2_InterpretingLogisticRegressionCoef.R)
   * [nes](https://github.com/stan-dev/stan/blob/feature/ARM/src/models/ARM/Ch.5/nes.stan): generalized linear model with logit link function and one predictor  
     ``vote ~ income``
  * [5.4 Logistic Regression Wells in Bangladesh](https://github.com/stan-dev/stan/blob/feature/ARM/src/models/ARM/Ch.5/5.4_LogisticRegressionWellsinBangladesh.R)
   * [wells](https://github.com/stan-dev/stan/blob/feature/ARM/src/models/ARM/Ch.5/wells_one_pred.stan): generalized linear model with logit link function and one predictor  
    ``switch ~ dist``
   * [wells](https://github.com/stan-dev/stan/blob/feature/ARM/src/models/ARM/Ch.5/wells_one_pred_scale.stan): generalized linear model with logit link function and one predictor  
     ``switch ~ dist100``
  * [5.5 Logistic Regression with Interactions](https://github.com/stan-dev/stan/blob/feature/ARM/src/models/ARM/Ch.5/5.5_LogisticRegressionWithInteractions.R)
   * [wells](https://github.com/stan-dev/stan/blob/feature/ARM/src/models/ARM/Ch.5/wells_interactions.stan): generalized linear model with logit link function and two predictors and interaction  
     ``switch ~ dist100 + arsenic + dist100:arsenic``
   * [wells](https://github.com/stan-dev/stan/blob/feature/ARM/src/models/ARM/Ch.5/wells_interactions_center.stan): generalized linear model with logit link function with two predictors and interaction centered using mean   
     ``switch ~ c_dist100 + c_arsenic + c_dist100:c_arsenic``
   * [wells](https://github.com/stan-dev/stan/blob/feature/ARM/src/models/ARM/Ch.5/wells_community.stan): generalized linear model with logit link function and four predictors and interaction centered using mean  
     ``switch ~ c_dist100 + c_arsenic + c_dist100:c_arsenic + assoc + educ4``
   * [wells](https://github.com/stan-dev/stan/blob/feature/ARM/src/models/ARM/Ch.5/wells_social.stan): generalized linear model with logit link function and three predictors and interaction centered using mean  
     ``switch ~ c_dist100 + c_arsenic + c_dist100:c_arsenic + educ4``
   * [wells](https://github.com/stan-dev/stan/blob/feature/ARM/src/models/ARM/Ch.5/wells_interactions_center_educ.stan): generalized linear model with logit link function and three predictors and interaction centered using mean  
     ``switch ~ c_dist100 + c_arsenic + c_educ4 + c_dist100:c_arsenic + c_dist100:c_educ4 + c_arsenic:c_educ4``
  * [5.6 Evaluating, Checking, & Comparing](https://github.com/stan-dev/stan/blob/feature/ARM/src/models/ARM/Ch.5/5.6_Evaluating%2CChecking%2C%26Comparing.R)
   * [wells](https://github.com/stan-dev/stan/blob/feature/ARM/src/models/ARM/Ch.5/wells_interactions_center_educ.stan): generalized linear model with logit link function with three predictors and interaction centered using mean  
     ``switch ~ c_dist100 + c_arsenic + c_educ4 + c_dist100:c_arsenic + c_dist100:c_educ4 + c_arsenic:c_educ4``
   * [wells](https://github.com/stan-dev/stan/blob/feature/ARM/src/models/ARM/Ch.5/wells_log_transform.stan): generalized linear model with logit link function with three predictors and interaction with log transform and centered using mean   
     ``switch ~ c_dist100 + c_log_arsenic + c_educ4 + c_dist100:c_log_arsenic + c_dist100:c_educ4 + c_log_arsenic:c_educ4``
   * [wells](https://github.com/stan-dev/stan/blob/feature/ARM/src/models/ARM/Ch.5/wells_log_transform2.stan): generalized linear model with logit link function with three predictors and interaction with log transform and centered using mean  
     ``switch ~ dist100 + log_arsenic + educ4 + dist100:log_arsenic + dist100:educ4 + log_arsenic:educ4``
  * [5.7 Average Predictive Comparisons](https://github.com/stan-dev/stan/blob/feature/ARM/src/models/ARM/Ch.5/5.7_AveragePredictiveComparisons.R)
   * [wells](https://github.com/stan-dev/stan/blob/feature/ARM/src/models/ARM/Ch.5/wells_edu.stan): generalized linear model with logit link function and three predictors  
     ``switch ~ dist100 + arsenic + educ4``
   * [wells](https://github.com/stan-dev/stan/blob/feature/ARM/src/models/ARM/Ch.5/wells_all.stan): generalized linear model with logit link function and three predictors with interaction  
     ``switch ~ dist100 + arsenic + educ4 + dist100:arsenic``
  * [5.8 Identifiability and Separating](https://github.com/stan-dev/stan/blob/feature/ARM/src/models/ARM/Ch.5/5.8_IdentifiabilityAndSeparation.R)
   * [simulation](https://github.com/stan-dev/stan/blob/feature/ARM/src/models/ARM/Ch.5/y_x.stan): generalized linear model with logit link function and one predictor  
     ``y ~ x``

***

#### [Chapter 6](https://github.com/stan-dev/stan/tree/feature/ARM/src/models/ARM/Ch.6)
  * [6.2 Poisson, Exposure, & Overdispersion - MISSING DATA](https://github.com/stan-dev/stan/blob/feature/ARM/src/models/ARM/Ch.6/6.2_Poisson%2CExposure%26Overdispersion.R)
  * [6.4 Probit Regression](https://github.com/stan-dev/stan/blob/feature/ARM/src/models/ARM/Ch.6/6.4_ProbitRegression.R)
   * [wells](https://github.com/stan-dev/stan/blob/feature/ARM/src/models/ARM/Ch.6/wells_probit.stan): generalized linear model with probit link function and one predictor  
    ``switch ~ dist100``
  * [6.5 Ordered & Unordered Categorical Regression - MISSING DATA](https://github.com/stan-dev/stan/blob/feature/ARM/src/models/ARM/Ch.6/6.5_Ordered%26UnorderedCategoricalRegression.R)
  * [6.7 More Complex GLM](https://github.com/stan-dev/stan/blob/feature/ARM/src/models/ARM/Ch.6/6.7_MoreComplexGLM.R)
   * [earnings](https://github.com/stan-dev/stan/blob/feature/ARM/src/models/ARM/Ch.6/earnings1.stan): generalized linear model with logit link function and two predictors  
     ``earn_pos ~ height + male`` 
   * [earnings](https://github.com/stan-dev/stan/blob/feature/ARM/src/models/ARM/Ch.6/earnings2.stan): linear model with two predictors and log transformation 
    ``log_earn ~ height + male, subset=earn>0``
  * [6.8 Constructive Choice Models](https://github.com/stan-dev/stan/blob/feature/ARM/src/models/ARM/Ch.6/6.8_ConstructiveChoiceModels.R)
   * [wells](https://github.com/stan-dev/stan/blob/feature/ARM/src/models/ARM/Ch.6/wells_logit.stan): generalized linear model with logit link function and one predictor  
    ``switch ~ dist100``

***

#### [Chapter 7](https://github.com/stan-dev/stan/tree/feature/ARM/src/models/ARM/Ch.7)
  * [7.1 Simulation of Probability Models](https://github.com/stan-dev/stan/blob/feature/ARM/src/models/ARM/Ch.7/7.1_SimulationOfProbabilityModels.R)
  * [7.2 Summarizing Linear Regression Using Simulation](https://github.com/stan-dev/stan/blob/feature/ARM/src/models/ARM/Ch.7/7.2_SummarizingLinearRegressionUsingSimulation.R)
   * [earnings](https://github.com/stan-dev/stan/blob/feature/ARM/src/models/ARM/Ch.7/earnings_interactions.stan): linear model with two predictors and interaction and log transformation   
    ``log_earn ~ height + male + height:male``
  * [7.3 Simulation for NonLinear Predictions](https://github.com/stan-dev/stan/blob/feature/ARM/src/models/ARM/Ch.7/7.3_SimulationForNonLinearPredictions.R)
   * [congress](https://github.com/stan-dev/stan/blob/feature/ARM/src/models/ARM/Ch.7/congress.stan): linear model with two predictors  
    ``vote_88 ~ vote_86 + incumbency_88``
  * [7.4 Predictive Simulation for GLM](https://github.com/stan-dev/stan/blob/feature/ARM/src/models/ARM/Ch.7/7.4_PredictiveSimulationForGLM.R)
   * [wells](https://github.com/stan-dev/stan/blob/feature/ARM/src/models/ARM/Ch.7/wells.stan): generalized linear model with logit link function and one predictor   
    ``switch ~ dist``
   * [earnings](https://github.com/stan-dev/stan/blob/feature/ARM/src/models/ARM/Ch.7/earnings1.stan): generalized linear model with logit link function and two predictors   
    ``earn_pos ~ height + male``
   * [earnings](https://github.com/stan-dev/stan/blob/feature/ARM/src/models/ARM/Ch.7/earnings2.stan): linear model with two predictors and log transformation 
    ``log_earn ~ height + male, subset=earn>0``

***

#### [Chapter 8](https://github.com/stan-dev/stan/tree/feature/ARM/src/models/ARM/Ch.8)
  * [8.1 Fake Data Simulation](https://github.com/stan-dev/stan/blob/feature/ARM/src/models/ARM/Ch.8/8.1_FakeDataSimulation.R)
   * [simulation](https://github.com/stan-dev/stan/blob/feature/ARM/src/models/ARM/Ch.8/y_x.stan): linear model with one predictor  
     ``y ~ x``  
  * [8.2 Fake Data Simulation to Understand Residual Plots](https://github.com/stan-dev/stan/blob/feature/ARM/src/models/ARM/Ch.8/8.2_FakeDataSimulationToUnderstandResidualPlots.R)
   * [grades](https://github.com/stan-dev/stan/blob/feature/ARM/src/models/ARM/Ch.8/grades.stan): linear model with one predictor  
     ``final ~ midterm``  
  * [8.3 Simulating from the Fitted Model - ADD ROACH MODELS STILL](https://github.com/stan-dev/stan/blob/feature/ARM/src/models/ARM/Ch.8/8.3_SimulatingFromTheFittedModel.R)
   * [lightspeed](https://github.com/stan-dev/stan/blob/feature/ARM/src/models/ARM/Ch.8/lightspeed.stan): linear model with no predictors  
     ``y ~ 1``  
  * [8.4 Predictive Simulation to Check Fit of Time Series Model](https://github.com/stan-dev/stan/blob/feature/ARM/src/models/ARM/Ch.8/8.4_PredictiveSimulationToCheckFitOfTimeSeriesModels.R)
   * [unemployment](https://github.com/stan-dev/stan/blob/feature/ARM/src/models/ARM/Ch.8/unemployment.stan): linear model with one predictor  
     ``y ~ y_lag``  

***

#### [Chapter 9](https://github.com/stan-dev/stan/tree/feature/ARM/src/models/ARM/Ch.9)
  * [9.3 Randomized Experiments - convert regression.2tables](https://github.com/stan-dev/stan/blob/feature/ARM/src/models/ARM/Ch.9/9.3_RandomizedExperiments.R)
   * [electric](https://github.com/stan-dev/stan/blob/feature/ARM/src/models/ARM/Ch.9/electric_one_pred.stan): linear model with one predictor  
     ``post_test ~ treatment``  
   * [electric](https://github.com/stan-dev/stan/blob/feature/ARM/src/models/ARM/Ch.9/electric_multi_preds.stan): linear model with two predictors  
     ``post_test ~ pre_test + treatment``  
  * [9.4 Treatment Interactions & Post Stratification](https://github.com/stan-dev/stan/blob/feature/ARM/src/models/ARM/Ch.9/9.4_TreatmentInteractions%26Poststratification.R)
   * [electric](https://github.com/stan-dev/stan/blob/feature/ARM/src/models/ARM/Ch.9/electric_one_pred.stan): linear model with one predictor    
     ``post_test ~ treatment``  
   * [electric](https://github.com/stan-dev/stan/blob/feature/ARM/src/models/ARM/Ch.9/electric_multi_preds.stan): linear model with two predictors  
     ``post_test ~ pre_test + treatment``  
   * [electric](https://github.com/stan-dev/stan/blob/feature/ARM/src/models/ARM/Ch.9/electric_interactions.stan): linear model with two predictors and interaction  
     ``post_test ~ pre_test + treatment + pre_test:treatment``  
  * [9.5 Observational Studies - havent started](https://github.com/stan-dev/stan/blob/feature/ARM/src/models/ARM/Ch.9/9.5_ObservationalStudies.R)

***

#### [Chapter 10](https://github.com/stan-dev/stan/tree/feature/ARM/src/models/ARM/Ch.10)
  * [10.3 Matching - MISSING DATA](https://github.com/stan-dev/stan/blob/feature/ARM/src/models/ARM/Ch.10/10.3_Matching.R)
  * [10.4 Lack of Overlap when Treat Assignments is Unknown](https://github.com/stan-dev/stan/blob/feature/ARM/src/models/ARM/Ch.10/10.4_LackOfOverlapWhenTreat.AssignmentIsUnknown.R)
   * [ideo](https://github.com/stan-dev/stan/blob/feature/ARM/src/models/ARM/Ch.10/ideo_two_pred.stan): linear model with two predictors  
    ``score1 ~ party + x, subset=overlap``
   * [ideo](https://github.com/stan-dev/stan/blob/feature/ARM/src/models/ARM/Ch.10/ideo_two_pred.stan): linear model with two predictors  
    ``score1 ~ party + x, subset=incs``
   * [ideo](https://github.com/stan-dev/stan/blob/feature/ARM/src/models/ARM/Ch.10/ideo_interactions.stan): linear model with two predictors and reparamaterization  
    ``score1 ~ party + I(z*(party==0)) + I(z*(party==1)), subset=incs``
   * [ideo](https://github.com/stan-dev/stan/blob/feature/ARM/src/models/ARM/Ch.10/ideo_reparam.stan): linear model with two predictors and interaction  
    ``score1 ~ party + x + party:x, subset=incs``
  * [10.5 Casual Effects Using IV](https://github.com/stan-dev/stan/blob/feature/ARM/src/models/ARM/Ch.10/10.5_CasualEffectsUsingIV.R)
   * [sesame](https://github.com/stan-dev/stan/blob/feature/ARM/src/models/ARM/Ch.10/sesame_one_pred_a.stan): linear model with one predictor  
    ``watched ~ encouraged``
   * [sesame](https://github.com/stan-dev/stan/blob/feature/ARM/src/models/ARM/Ch.10/sesame_one_pred_b.stan): linear model with one predictor  
    ``y ~ encouraged``
  * [10.6 IV in a Regression Framework - misisng stuff at bottom](https://github.com/stan-dev/stan/blob/feature/ARM/src/models/ARM/Ch.10/10.6_IVinaRegressionFramework.R)
   * [sesame](https://github.com/stan-dev/stan/blob/feature/ARM/src/models/ARM/Ch.10/sesame_one_pred_a.stan): linear model with one predictor  
    ``watched ~ encouraged``
   * [sesame](https://github.com/stan-dev/stan/blob/feature/ARM/src/models/ARM/Ch.10/sesame_one_pred_2b.stan): linear model with one predictor  
    ``y ~ watched_hat``
   * [sesame](https://github.com/stan-dev/stan/blob/feature/ARM/src/models/ARM/Ch.10/sesame_multi_preds_3a.stan): linear model with three predictors and one factor  
    ``watched ~ encouraged + pretest + as.factor(site) + setting``
   * [sesame](https://github.com/stan-dev/stan/blob/feature/ARM/src/models/ARM/Ch.10/sesame_multi_preds_3b.stan): linear model with three predictors and one factor  
    ``y ~ watched_hat + pretest + as.factor(site) + setting``

***

#### [Chapter 11](https://github.com/stan-dev/stan/tree/feature/ARM/src/models/ARM/Ch.11)
  * [11.3 Repeated Measurements, Time-Series, Cross Sections & Others - missing data](https://github.com/stan-dev/stan/blob/feature/ARM/src/models/ARM/Ch.11/11.3_RepeatedMeasurements%2CTimeSeries%20CrossSections%26Others.R)

***

#### [Chapter 12](https://github.com/stan-dev/stan/tree/feature/ARM/src/models/ARM/Ch.12)
  * [12.2 Partial Pooling with No Predictors](https://github.com/stan-dev/stan/blob/feature/ARM/src/models/ARM/Ch.12/12.2_PartialPoolingWithNoPredictors.R)
    * [radon_intercept](https://github.com/stan-dev/stan/blob/feature/ARM/src/models/ARM/Ch.12/radon_intercept.stan): multi-level linear model with varying intercept        
      ``lmer (y ~ 1 + (1 | county))``
  * [12.3 Partial Pooling with Predictors](https://github.com/stan-dev/stan/blob/feature/ARM/src/models/ARM/Ch.12/12.3_PartialPoolingWithPredictors.R) 
    * [radon_complete_pool](https://github.com/stan-dev/stan/blob/feature/ARM/src/models/ARM/Ch.12/radon_complete_pool.stan): multi-level linear model with complete pooling          
      ``lm (y ~ x)``
    * [radon_no_pool](https://github.com/stan-dev/stan/blob/feature/ARM/src/models/ARM/Ch.12/radon_no_pool.stan): multi-level linear model with no pooling        
      ``lmer (y ~ x + (1 | county))``
  * [12.4 Fitting MLM in R](https://github.com/stan-dev/stan/blob/feature/ARM/src/models/ARM/Ch.12/12.4_FittingMLMinR.R)
    * [radon_intercept](https://github.com/stan-dev/stan/blob/feature/ARM/src/models/ARM/Ch.12/radon_intercept.stan): multi-level linear model with varying intercept         
      ``lmer (y ~ 1 + (1 | county))``
    * [radon_no_pool](https://github.com/stan-dev/stan/blob/feature/ARM/src/models/ARM/Ch.12/radon_no_pool.stan): multi-level linear model with no pooling          
      ``lmer (y ~ x + (1 | county))``        
  * [12.6 Group-Level Predictors](https://github.com/stan-dev/stan/blob/feature/ARM/src/models/ARM/Ch.12/12.6_Group-LevelPredictors.R)
    * [radon_group](https://github.com/stan-dev/stan/blob/feature/ARM/src/models/ARM/Ch.12/radon_group.stan): multi-level linear model with group level predictor and individual level predictors              
      ``lmer (y ~ x + u + (1 | county))``
    * [radon_no_pool](https://github.com/stan-dev/stan/blob/feature/ARM/src/models/ARM/Ch.12/radon_no_pool.stan): multi-level linear model with no pooling                
      ``lmer (y ~ x + (1 | county))``
  * [12.8 Prediction](https://github.com/stan-dev/stan/blob/feature/ARM/src/models/ARM/Ch.12/12.8_Prediction.R)
    * [radon_group](https://github.com/stan-dev/stan/blob/feature/ARM/src/models/ARM/Ch.12/radon_group.stan): multi-level linear model with group level predictor and individual level predictors         
      ``lmer (y ~ x + u + (1 | county))``

***

#### [Chapter 13](https://github.com/stan-dev/stan/tree/feature/ARM/src/models/ARM/Ch.13)

  * [13.1 Varying Intercepts & Slopes](https://github.com/stan-dev/stan/blob/feature/ARM/src/models/ARM/Ch.13/13.1_VaryingIntercepts%26Slopes.R)

    * [radon_vary_si](https://github.com/stan-dev/stan/blob/feature/ARM/src/models/ARM/Ch.13/radon_vary_si.stan): multi-level linear model with group level predictors         
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

  * [13.4 Understanding Correlations Between Intercepts & Slopes](https://github.com/stan-dev/stan/blob/feature/ARM/src/models/ARM/Ch.13/13.4_UnderstandingCorrelationsBetweenIntercepts%26Slopes.R)
 
   * [earnings_vary_si](https://github.com/stan-dev/stan/blob/feature/ARM/src/models/ARM/Ch.13/earnings_vary_si.stan): multi-level linear model with group level predictors         
```
lmer (y ~ x (1 + x | ethn))
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

#### [Chapter 14](https://github.com/stan-dev/stan/tree/feature/ARM/src/models/ARM/Ch.14)
 
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

#### [Chapter 15](https://github.com/stan-dev/stan/tree/feature/ARM/src/models/ARM/Ch.15)

***

THE BELOW CHAPTERS HAVE NOT BEEN ADDED YET.

#### [Chapter 16](https://github.com/stan-dev/stan/tree/feature/ARM/src/models/ARM/Ch.16)

#### [Chapter 17](https://github.com/stan-dev/stan/tree/feature/ARM/src/models/ARM/Ch.17)

#### [Chapter 18](https://github.com/stan-dev/stan/tree/feature/ARM/src/models/ARM/Ch.18)

#### [Chapter 19](https://github.com/stan-dev/stan/tree/feature/ARM/src/models/ARM/Ch.19)

#### [Chapter 20](https://github.com/stan-dev/stan/tree/feature/ARM/src/models/ARM/Ch.20)

#### [Chapter 21](https://github.com/stan-dev/stan/tree/feature/ARM/src/models/ARM/Ch.21)

#### [Chapter 22](https://github.com/stan-dev/stan/tree/feature/ARM/src/models/ARM/Ch.22)

#### [Chapter 23](https://github.com/stan-dev/stan/tree/feature/ARM/src/models/ARM/Ch.23)

#### [Chapter 24](https://github.com/stan-dev/stan/tree/feature/ARM/src/models/ARM/Ch.24)

#### [Chapter 25](https://github.com/stan-dev/stan/tree/feature/ARM/src/models/ARM/Ch.25)