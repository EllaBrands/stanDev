This is a place where developers and users can pool ideas on different ways to facilitate the use of Stan for complex Ordinary Differential Equations (ODE) based models. 

# Introduction
An ODE based models describes the evolution of certain quantities of interest over time. Our goal is to compute these quantities at the various events of an event schedule. An *event* corresponds to either a measurement or a change in the state of the system. If the system is fully described by ODE, knowing the state y0 at time t0 fully defines the solution at finite times, provided there are no exterior changes in the state of the system. Exploiting this property, we can sequentially calculate the quantities of interest one event at a time, if we set the initial conditions at the current event to be the quantities at the previous event. For this strategy to be successful, we need:

1. a solution operator that solves the ODE system at any one particular event
2. an event schedule that includes all the events that alter the state of the system 

ODE-based models have applications in multiple fields, such as pharmacometrics, system biology, and biogeochemistry. Each fields has its own requirements and conventions. Hence one of the challenges we face is to identify what is generalizable versus what needs to be tailored to each field. As far as Stan development goes, this taps into the broader question of how to implement domain-specific modules.  

# Challenges

* Solve systems of ODEs
  * stiff and non-stiff
  * optimization methods (for example, for linear systems)
  * analytical solutions (when possible)

* Event Schedule Handler  
  * data format (likely varies from one field to the other, so will need a translator from one format to the other)
  * discontinuity in the data
  * reset events 
  * steady state approximations 

* Integrate functions and expressions in Stan
  * generalizable procedures 
  * domain specific procedures  

## Solution Operator

Users should be able:

a. write the solution operator in the function block of stan
b. pick an ODE integrator 
c. call a built-in analytical operator

Note that a solution operator may apply only to certain kind of events. For example, I had to write unique operators for Steady State approximations for the one a two compartment pharmacokinetic models. 

#### Implemented
* non-stiff ODE solver: Runge-Kutta 4th/5th (rk45)
* stiff ODE solver: CVODES (bdf)

#### Not Implemented
* Solver for linear systems using matrix exponentials
* Adaptive method that recognizes when ODE is in a stiff and non-stiff region, and switches between the bdf and rk45 methods
* Analytical solutions for compartment models
  * extensively used in pharmacometrics, but also seems to have applications in other fields (see soil metamodel: https://github.com/soil-metamodel/stan/tree/master/soil-incubation)
* Solver for Steady State approximation (boundary value problem)
  * Root solver
  * IDAS from Sundials (?)
* Parallelization ODE solving (for multiple patients -- population models)
  
## Event Handler 

Event Handler should be able to:
(a) augment the schedule so that it contains all the events that alter the state of the system
(b) apply a solution operator to each event of the augmented event schedule
(c) return the predicted quantities for at all events of the original event schedule

State Changers: 
* discontinuous change in quantity (bolus dosing)
* continuous change in quantity (infusion)
* reset event (set all quantities of interest to 0)
* steady state approximation
* change in parameters of the ODE system
  * time-dependent parameters 
  * variation within levels (inter-individual, inter-study, ...)

The solution operator may change from one event to the other (steady state approximation may require a different operator for efficient computation). 

We could split the Event Handler into two functions that respectively handle (a), and (b) + (c). The advantage of doing so is that often times the Event Schedule only involves data and the augmenting of the event schedule can thus take place in the transformed data block. (b) usually handles model parameters, used in the ODE system. 

## Domain Specific Applications

### Pharmacometrics

Needs: 
* compartment models
* the ability to input an event schedule that follows NONMEM conventions
* functions that return amounts in each compartment at each event


#### Torsten 
(prototype version 0.8: pharmacometrics specific implementation)

Four new Stan expressions for pharmacometrics applications are programmed in C++. They are integrated within Stan so that they may be used in a manner identical to built-in Stan functions. The expressions predict amounts in a compartment model, given an event schedule. The input data follows NONMEM conventions. The current prototype contains analytical solutions to one and two compartment models with first-order absorption (including solutions for steady state approximations), and numerical solutions to ODE based models. In the latter case, the user can specify the ODE system in the function block of a Stan file. 

At a C++ level, all the exposed functions call the same function Pred, which acts as the *event handler*, i.e it augments the event schedule and applies a solution operator to the ODE system at each event. The solution operators Pred1, or PredSS for steady state approximations, predicts amounts for one event, and varies from one expression to the other. This structural scheme is meant to make Torsten easily expandable: to add a new function, a developer only needs to focus on how to predict amounts for one event, while the rest is handled by the Pred function. 

Example code is available on GitHub: https://github.com/charlesm93/example-models/tree/feature/issue-70-PKPDexamples-torsten/PKPD/torsten

Specifically, I recommend looking at the `torstenManual.pdf` for information on how to use Torsten, run the example code, as well as some information on the design. 


