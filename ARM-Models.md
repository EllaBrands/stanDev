Links to the ARM models will be posted when they are available.

### Organized by Chapter
#### Chapter 2
 * [2.3 CI](https://github.com/stan-dev/stan/blob/feature/ARM/src/models/ARM/Ch.2/2.3_CI.R)
 * [2.4 Hypothesis Testing - Missing Data](https://github.com/stan-dev/stan/blob/feature/ARM/src/models/ARM/Ch.2/2.4_HypothesisTesting.R)
 * [2.6 55,000 Residents - Missing Data](https://github.com/stan-dev/stan/blob/feature/ARM/src/models/ARM/Ch.2/2.6_55%2C000Residents.R)

#### Chapter 3
 * [3.1 One Predictor](https://github.com/stan-dev/stan/blob/feature/ARM/src/models/ARM/Ch.3/3.1_OnePredictor.R)
   * kid_iq: linear model with one predictor (kid_score ~ mom_hs)
   * kid_iq: linear model with one predictor (kid_score ~ mom_iq)
 * [3.2 Multiple Predictors](https://github.com/stan-dev/stan/blob/feature/ARM/src/models/ARM/Ch.3/3.2_MultiplePredictors.R)
   * kid_iq: linear model with two predictors (kid_score ~ mom_hs + mom_iq)
 * [3.3 Interactions](https://github.com/stan-dev/stan/blob/feature/ARM/src/models/ARM/Ch.3/3.3_Interactions.R)
   * kid_iq: linear model with two predictors and interaction (kid_score ~ mom_hs + mom_iq + mom_hs:mom_iq)
 * [3.4 Stat Inference](https://github.com/stan-dev/stan/blob/feature/ARM/src/models/ARM/Ch.3/3.4_StatInference.R)
   * kid_iq: linear model with two predictors (kid_score ~ mom_hs + mom_iq)
 * [3.5 Graph Displays](https://github.com/stan-dev/stan/blob/feature/ARM/src/models/ARM/Ch.3/3.5_GraphDisplays.R)
   * kid_iq: linear model with one predictor (kid_score ~ mom_iq)
   * kid_iq: linear model with two predictors (kid_score ~ mom_hs + mom_iq)
   * kid_iq: linear model with two predictors and interaction (kid_score ~ mom_hs + mom_iq + mom_hs:mom_iq)
 * [3.6 Diagnostics](https://github.com/stan-dev/stan/blob/feature/ARM/src/models/ARM/Ch.3/3.6_Diagnostics.R)
   * kid_iq: linear model with one predictor (kid_score ~ mom_iq)
 * [3.7 Prediction](https://github.com/stan-dev/stan/blob/feature/ARM/src/models/ARM/Ch.3/3.7_Prediction.R)
   * kid_iq: linear model with two predictors (kid_score ~ mom_hs + mom_iq)
   * kid_iq: linear model with two predictors (ppvt ~ hs + afqt)

#### Chapter 4
  * 4.1 Linear Transformations
  * 4.2 Centering & Standardizing
  * 4.4 Log Transformations
  * 4.5 Other Transformations
  * 4.6 Regression Models for Prediction
  * 4.7 Fitting a Series of Regressions - not attempted yet

#### Chapter 5
  * 5.1 Logistic Regression with One Predictor
  * 5.2 Interpreting Logistic Regression Coeficients
  * 5.4 Logistic Regression Wells in Bangladesh
  * 5.5 Logistic Regression with Interactions
  * 5.6 Evaluating, Checking, & Comparing
  * 5.7 Average Predictive Comparisons
  * 5.8 Identifiability and Separating

#### Chapter 6
  * 6.2 Poisson, Exposure, & Overdispersion - Missing data
  * 6.4 Probit Regression
  * 6.5 Ordered & Unordered Categorical Regression - Missing Data
  * 6.6 More Complex GLM
  * 6.7 Constructive Choice Models

#### Chapter 7
  * 7.1 Simulation of Probability Models
  * 7.2 Summarizing Linear Regression Using Simulation - predict?
  * 7.3 Simulation for NonLinear Predictions
  * 7.4 Predictive Simulation for GLM

#### Chapter 8
  * 8.1 Fake Data Simulation
  * 8.2 Fake Data Simulation to Understand Residual Plots
  * 8.3 Simulating from the Fitted Model - add roach models
  * 8.4 Predictive Simulation to Check Fit of Time Series Model

#### Chapter 9
  * 9.3 Randomized Experiments - convert regression.2tables
  * 9.4 Treatment Interactions & Post Stratification
  * 9.5 Observational Studies - havent started

#### Chapter 10
  * 10.3 Matching - missing data
  * 10.4 Lack of Overlap when Treat Assignments is Unknown
  * 10.5 Casual Effects Using IV
  * 10.6 IV in a Regression Framework - misisng stuff at bottom

#### Chapter 11
  * 11.3 Repeated Measurements, Time-Series, Cross Sections & Others - missing data