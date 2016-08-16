# Terminology

**System Model**: A model that describes the evolution of a system over time. 

**Event**: Either the observation of a certain quantity or a change in the state of the system:  

* **Observation Event**: usually specifies the time and the quantity of interest at which we want to simulate data
* **State Changer**: An (exterior) intervention that alters the state of the system 

**Evolution Operator**: Takes in the state of a system at time t0 and returns the state of that system at time t0 + t, provided that:

* knowing the state at time t0 fully defines the state at finite times
* between t0 and t0 + t, the system is isolated, i.e. there is no exterior intervention that alters the state of the system

**Event Handler**: Takes in a schedule of events and returns the quantities of interest at each event.  



# Introduction
A system model describes the evolution of certain quantities of interest over time. Our goal is to compute these quantities at the various events of an event schedule. Often times, a system model is based on Ordinary Differential Equations (ODEs), which we might expect, since ODEs describe the rate at which quantities change. If the system is fully described by ODEs, knowing the state y0 at time t0 fully defines the solution at finite times, provided there are no exterior changes in the state of the system. Exploiting this property, we can sequentially calculate the quantities of interest one event at a time, if we set the initial conditions at the current event to be the quantities at the previous event. For this strategy to be successful, we need:

1. an evolution operator that evolves the system between two time points
2. an event schedule that includes all the events that alter the state of the system 

System models have applications in multiple fields, such as pharmacometrics, system biology, and biogeochemistry. Each fields has its own requirements and conventions. Hence one of the challenges we face is to identify what is generalizable versus what needs to be tailored to each field.  

# Challenges

* Evolution Operators
  * often involves solving ODE systems
  * analytical solutions can represent many lines of code

* Event Schedule Handler  
  * data format for events (likely varies from one field to the other, so will need a translator from one format to the other)
  * discontinuity in the data due to State Changers
  * steady state approximations 

* Integrate functions and expressions in Stan
  * generalizable procedures 
  * domain specific procedures  

## Evolution Operator

Users should be able:

1. write the evolution operator in the function block of stan
2. call a built-in analytical operator

An evolution operator may apply only to certain kind of events. For example, I had to write unique operators for Steady State approximations for the one and two compartment pharmacokinetic models. 

In many cases, a good evolution operator requires a good way to solve ODE systems, so I provide an overview of available tools, and tools we could consider implementing: 

#### Implemented
* non-stiff ODE solver: Runge-Kutta 4th/5th (rk45)
* stiff ODE solver: CVODES (bdf)

#### Not Implemented
* Adaptive method that recognizes when ODE is in a stiff and non-stiff region, and switches between the bdf and rk45 methods
* Additional ODE solvers
  * Matrix Exponential (for linear systems)
  * Adams Moulton
* Analytical solutions for compartment models
  * extensively used in pharmacometrics, but also seems to have applications in other fields (see soil metamodel: https://github.com/soil-metamodel/stan/tree/master/soil-incubation)
* Solver for Steady State approximation (boundary value problem)
  * Root solver
  * IDAS from Sundials (?)
  * Numerical solution of algebraic equations
* Parallelization ODE solving (for multiple patients -- population models)

  
## Event Handler 

Event Handler should be able to:

1. augment the schedule so that it contains all the events that alter the state of the system
2. compute changes due to exterior interventions
3. apply an evolution operator to certain events
4. return the predicted quantities for at all events of the original event schedule

The first step may or may not be required depending on the format of the event schedule. In pharmacometrics, certain events can correspond to multiple events. For example, the administration of five doses at a regular interval will be recorded as one event but really corresponds to five events. The event handler would convert this one event into five single-dose events, thereby calculating the times at which the state of the system is altered. 

This step is crutial because we cannot apply an evolution operator between t0 and t1, if at a time t2 between those two points the state of the system is altered. Rather, we must apply the evolution operator between t0 and t2, compute the dosing, and then apply the evolution operator from t2 to t1. 

This naturally brings us to step 2. The evolution of the system is not entirely determined by the evolution operator, and the event handler must compute other changes. In the dosing example, this corresponds to a discontinuous change in one of the quantities.

Whether (2) or (3) occurs depends on the event. 

I now list a few events the event handler should be able to deal with. Examples from pharmacometrics are written in parentheses.  

State Changers: 
* discontinuous change in quantity (bolus dosing)
* continuous change in quantity (rate)
* reset event: set all quantities of interest to 0
* steady state approximation
* change in parameters (time-dependent parameters) 
* change in input function
  * change in constant rate input for a quantity (change in input rate for one compartment) 
* Lag time: Shifts the time at which state changers occur (in pharmacometrics, lag times are usually compartment specific, meaning they only affect one quantity. They can however be multiple lag time parameters)

Additionally, the evolution operator may change from one event to the other (steady state approximations may require a different operator for efficient computation), and the event handler needs to recognize which operator to apply. 

We could split the Event Handler into two functions that respectively handle (1), and (2) + (3) + (4). The advantage of doing so is that often times the Event Schedule only involves data and the augmenting of the event schedule can thus take place in the transformed data block. 

## Domain Specific Applications

### Pharmacometrics

Needs: 
* compartment models
* the ability to input an event schedule that follows NONMEM conventions
* functions that return amounts in each compartment at each event

There are currently two prototype libraries on GitHub aimed at facilitating Stan for PKPD modeling: The **PKPD Library** and **Torsten**. These prototypes offer two different approaches: the PKPD Library comprises Stan files and R scripts, while Torsten is a set of procedures directly coded in C++. Authors of both projects are working together towards an optimal approach. 

In addition, the R package **PMX Stan** also supports Bayesian PKPD modeling in Stan (http://andrewgelman.com/2015/10/05/pmxstan-an-r-package-to-facilitate-bayesian-pkpd-modeling-with-stan/). 

#### Draft of PKPD Library 
The files in this library are an attempt to ease writing PKPD models in Stan. Pharmacokinetic (PK) models describe the relationship of drug concentration in a given subject over time as a function of the dosing history. PK models facilitate mass action kinetics laws to describe the drug absorption, distribution and elimination process with a compartmental approximation.

https://github.com/stan-dev/example-models/tree/feature/issue-72-stan-pkpdlib/misc/pkpd


#### Torsten: A PKPD Model Library for Stan 

Four new Stan functions for pharmacometrics applications are programmed in C++. They are integrated within Stan so that they may be used in a manner identical to built-in Stan functions. These functions predict amounts in a compartment model, given an event schedule. The input data follows NONMEM conventions. The current prototype contains analytical solutions to one and two compartment models with first-order absorption (including solutions for steady state approximations), and numerical solutions to ODE based models. In the latter case, the user must specify the ODE system in the function block of a Stan file. 

At a C++ level, all the exposed functions call the same function Pred, which acts as the *event handler*, i.e it augments the event schedule and applies the evolution operator. The evolution operator Pred1, or PredSS for steady state approximations, predicts amounts for one event, and varies from one expression to the other. This structural scheme is meant to make Torsten easily expandable: to add a new function, a developer only needs to focus on how to predict amounts for one event, while the rest is handled by the Pred function. 

https://github.com/charlesm93/example-models/tree/feature/issue-70-PKPDexamples-torsten/PKPD/torsten

Specifically, I recommend looking at the `torstenManual.pdf` for information on how to use Torsten, run the example code, as well as some discussion of the design. 



## Implementation Example: Torsten

Let's take a look at some code from Torsten to illustrate the concepts discussed above. 

The stan code can be found in the above link and the C++ code is available here: https://github.com/charlesm93/math/tree/feature/issue-314-torsten-pharmacometrics/stan/math/torsten

We'll focus on `TwoCptModelExample.stan`, a pharmacokinetics two compartment model. Mathematically, the model can be described by an ODE system that tells us how the amount of an administered drug evolves in different parts of the body or *compartments* (gut, blood, ...). 


The example code calls the Torsten function `PKModelTwoCpt`: 

```
  # PKModelTwoCpt takes in the parameters matrix and the NONMEM data,
  # and returns a matrix with the predicted amount in each compartment 
  # at each event.  
  x = PKModelTwoCpt(theta, time, amt, rate, ii, evid, cmt, addl, ss);
```

`theta` gives the parameters that will be used for the evolution operator, while `time, amt, rate, ii, evid, cmt, addl, ss` together comprise the event schedule, where I have followed NONMEM conventions. 

At a C++ level, *PKModelTwoCpt* calls Torsten's event handler, *pred*, and passes the parameters and event schedule to it. But before doing so, *PKModelTwoCpt* defines the evolution operators. For pharmacometrics models, we typically have two such operators: *pred1* (regular operator), and *predSS* (steady state operator, which we will not worry about in this example).

Evolution operators in Torsten are functors. The constructor recognizes the key words "TwoCptModel" and defines an operator that calls a function stored in `PKModel/pred/pred1_two.hpp`. 

```
  //Construct Pred functions for the model.
  Pred1_structure new_Pred1("TwoCptModel");
  PredSS_structure new_PredSS("TwoCptModel");
  Pred1 = new_Pred1;
  PredSS = new_PredSS;
        
  Matrix <typename promote_args<T0, T1, T2, T3, T4>::type, Dynamic, Dynamic> pred;
  pred = Pred(pMatrix, time, amt, rate, ii, evid, cmt, addl, ss, model, dummy_ode());

  return pred;
```

The evolution operator is not passed in as an argument in Stan; rather by picking `PKModelTwoCpt` the user chooses the evolution operator. All four torsten functions are essentially identical, but construct different evolution operators. `generalCptModel_*` functions require the users to pass in an ODE system and tuning parameters for the ODE integrator their evolution operators call.

The event handler *pred* is defined in `PKModel/pred.hpp`. It first augments the event schedule and then predicts the quantities in the system. 

Augmenting the event schedule requires understanding the NONMEM data structure; it is tedious, domain specific, and not very profound; which is why I won't discuss it here. 

I'll jump ahead to predicting quantities, and will focus on one section of the code:

```
	for(i=0;i<events.get_size();i++)
	{
		event = events.GetEvent(i);
		
		[...]
		
		if((event.evid == 3)||(event.evid == 4)) //reset events
		{		
			dt=0;
			init = zeros;
			
		}

		
		else
		{
			dt = event.time - tprev;
			pred1 = Pred1(dt, parameter, init, rate2.rate, f);
			init = pred1;
		}
		
		[...]

```

*pred* goes through each event of the augmented event schedule and performs various operations depending on the event type. The *evid*, which stands for *event identity*, tells *pred* what to do. During a reset event, *pred* does not apply the evolution operator, but simply sets all the quantities to zero. The quantities are stored in the vector *init*. I use the name *init* because the quantities at the current event usually become the initial quantities at the next event. Computing quantities is a matter of updating the *init* vector. 

More interestingly, *pred* usually calls the evolution operator *pred1*. Note that the time of the event itself doesn't matter, what matters is the time since the system was in the initial state. *f* has no use here but other evolution operators sometimes require an ODE system, which is stored in *f*. 

Finally, when *pred* has gone through all the events, it returns the quantities at each event. The final output only has quantities from the original event schedule, not the augmented one. 


## Improving from Torsten's Design

Goal:
* provide a more general framework for event handling

Instead of having four built-in functions, I'd want one event handler function with the following signature: 

```
  matrix pred(pred1, parameters, events);
```

where pred1 is the evolution operator. Not sure how to handle the case where multiple evolution operators are required. 

The user specifies `pred1` with the following signature: 

```
  vector pred1(dt, y0, parameter, data, f);
```
 
where dt is the time to the previous event, and y0 the initial state of the system. Not sure what constraints to put on *parameter* and *data*, but in Torsten, parameter is simply the model parameters, and data rates. *f* is for passing ODE system, to address the case the user would want his evolution operator to use an ODE integrator. 

I realize Stan doesn't allow higher-order functions yet, which would be a problem both for the evolution operator and the event handler. 

In addition, I would want a few built-in functions (either in Stan or in an extention) for frequently used evolution operators. The analytical solution to the Two Comparment System is a good example. The user could then pass:

```
  x = pred(pred1_twoCpt, parameters, events);
```

I'm wondering if we should require *events* to be augmented before passing them into *pred*. We could then have functions that augment the event schedule data and convert it into something readily usable for *pred* when it computes the quantities. 
