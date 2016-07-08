This is a place where developers and users can pool ideas on different ways to facilitate the use of Stan in Pharmacometrics, and other fields that present similar challenges. 

## Challenges
* Solve non-linear systems of ODEs
* Handle event schedule (of clinical trials)
* Create template and functions that make coding efficient 
* Make code easy to install and maintain 

## Approaches 

### Torsten 
(prototype version 0.8)

New Stan functions for pharmacometrics applications are programmed in C++. They are integrated within Stan so that the new functions may be used in a manner identical to built-in Stan functions. The functions predict amounts in a compartment model, given an event schedule. The input data follows NONMEM conventions. The current prototype contains analytical solutions to one and two compartment models, and numerical solutions to ODE based models (the user can specify the ODE system in the function block of a Stan file). 

This method allows elaborate pharmacometrics procedures to be efficiently coded in Stan. The possibility to work directly in Stan should also give the user much flexibility.  

At a C++ level, all the exposed functions call the same function Pred, which handles the event schedule and predicts the amount in each compartment for all events. On the other hand Pred1, which predicts amounts for one event, varies from one function to the other. This structural scheme is meant to make Torsten easily expendable: to add a new function, a developer only needs to focus on how to predict amounts for one event, while the rest is handled by the Pred function. 

