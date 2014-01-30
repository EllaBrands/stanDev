## Overview

We are going to add support for ordinary differential equation (ODE) solvers (i.e., integrators) to Stan for stiff and non-stiff systems.  The language will be extended to allow ODEs to be defined in Stan and then used for solution.  We want to support both easy and stiff ODEs and would like to provide error control on both the ODE solutions and their gradients with respect to system parameters.

### Auto-diff the System Integrator vs. Integrate the System Auto-diff

If there are parameters involved in the definition of the differential equation, Stan needs the derivatives of the solutions to the system with respect to the parameters.  There are two approaches to doing this.  

1.  Automatically differentiate the integrator. 

2.   Differentiate a coupled system consisting of the original state variables coupled with derivatives of the state variables with respect to the parameters. 

The derivatives required for the coupled system could themselves be evaluated by 

### Handling Stiff ODEs

In order to handle stiff differential equations, the Jacobian of the system of differential equations is required.  This can be done for either of the two approaches discussed in the previous section.  In the coupled case, the Jacobian of the coupled system could be calculated by a higher-order automatic differentiation.

## Example: Harmonic Oscillator

The harmonic oscillator is a two-variable system of differential equations, for which we'll include a single parameter `g`.  The system involves a two-dimensional state vector `x` and is defined by the following pair of ordinary differential equations.

```
d/dt x[0] = x[1]
d/dt x[1] = -x[0] - g * x[1]
```

The coupled system adds two additional state variables `x[2]` and `x[3]` defined as the derivatives of the original state variables `x[0]` and `x[1]` with respect to the parameter `g`:

```
x[2] = d/dg x[0]
x[3] = d/dg x[1]
```

To complete the coupled systems, we need the derivatives with respect to time,

```
d/dt x[2] = d/dt d/dg x[0]
          = d/dg d/dt x[0]
          = d/dg x[1] 
          = x[3]

d/dt x[3] = d/dt d/dg x[1]
          = d/dg d/dt x[1]
          = -d/dg x[0] - d/dg (g * x[1])
          = -x[2] - ( (d/dg g) * x[1] + g * d/dg x[1] )
          = -x[2] - x[1] - g * x[3]
```

By integrating the coupled system, the solutions for `x[0], x[1]` are the original solutions and the solutiosn for `x[2], x[3]` provide derivatives of the solutions with respect to the parameter `g`.  This allows the integrator to be run using only `double` types and also provides tolerance control for the derivatives.


## Working Code Branch

There's a Stan GitHub branch Daniel and I started for experimentation with integrators:

```
feature/ode_experiment
```

It currently contains a fully working example of Stan fitting the parameter for a harmonic oscillator given noisy measurements.  

The code is based on Boost's odeint package:

* <a href="http://headmyshoulder.github.io/odeint-v2/">odeint home page</a>
* <a href="http://www.boost.org/doc/libs/1_55_0/libs/numeric/odeint/doc/html/index.html">Boost's odeint documentation</a>

The harmonic oscillator is based on the

* <a href="http://www.boost.org/doc/libs/1_55_0/libs/numeric/odeint/doc/html/boost_numeric_odeint/tutorial/harmonic_oscillator.html">Boost harmonic oscillator example</a>

The simple auto-diff of the integrator approach is in:

```
src/stan/math/functions/ho.hpp
```

The integration of the coupled system including the derivatives is in:

```
src/stan/math/functions/ho2.hpp
```

The latter approach, `ho2.hpp`, is about 5 times as efficient, and also provides error control.  

The issue we have now is that we can't define this coupled system by reverse-mode auto-diff because we only have a single auto-diff stack.  This problem can be overcome by just using the top of the auto-diff stack, but that will require us to enhance the derivative propagation algorithm to take in a stack position at which to stop the propagation.

The system definition in the example creates a function `harmonic_oscillator` that is available just like any other function in a Stan model.  The model we use assumes measurements of a system with normal noise with scale `sigma`:

```
data {
  int<lower=0> N;
  int<lower=1> M;
  real t[N];
  matrix[N,M] x_obs;
  vector[M] x0;
}
parameters {
  real<lower=-10,upper=10> g;
  real<lower=0, upper=1> sigma[2];
}
model {
  matrix[N,M] x_hat;
  x_hat <- harmonic_oscillator(t, x0, g);
  for (n in 1:N)
    x_obs[n] ~ normal(x_hat[n], sigma);
}
```

The ODE defines a function ```harmonic_oscillator``` that takes an array of times and returns an array `x_hat` of values at those times.  The values in the array are of the same type as the state and the array has the same size (number of entries) as the time argument ```t```.  The value ```x0``` is the initial state for the ODE.  The argument `g` is for the parameter;  in general, there may be more than one parameter.  Note also that the parameter ```g``` here is also a parameter to the Stan model, meaning it's something we need to differentiate with respect to.  This is all handled in the definition of the function in the files `ho.hpp` or `ho2.hpp`.

## Stan Language for Defining Systems of Differential Equations

This section lists a few possible ways in which differential equations could be expressed in Stan's modeling language.  We were inspired by Frederic Bois's GNU package <a href="http://en.wikipedia.org/wiki/MCSim">MCSim</a>.

### Proposal 1: Stan-like System Syntax

#### Defining the System

A natural Stan-like way of defining a system of differentiation equations would be to allow the state variables to be anything.  For a simple vector state, this would look like:

```
differential equations {
  harmonic_oscillator {
    state {
      vector[2] x;
    }
    parameters {
      real<lower=0> gamma;
    }
    dynamics {
      d.x[1] <- x[2];
      d.x[2] <- -x[1] - gamma * x[2];
    }
}
```

The `state` block defines the state, the `parameters` block the parameters, and there would also need to be a `data` block to define constants.  

The `dynamics` block is required to define derivatives with respect to a time variable `t` for each state variable.  The special variables `d.x[1]` would be implicitly defined with the same basic types and dimensions as the state variables.  The special variable `t` for time would also be implicitly defined in the `dynamics` block.  

#### Calling the Integrator

Given initial values for the state variables (the coupled system's initial values can be calculated automatically), and given a set of time points, we need a function that returns the states that form solutions.  

This is tricky for this formulation because it's not clear what the return type should be.  One approach would be to allow mutable references.  For example,

```
ode_solve(harmonic_oscillator,       // system
          vector[2] x_0,             // initial state at t=0
          real times[K],             // times for solutions
          real g,                    // parameter
          vector[2] solutions[K]);   // solutions returned
```

Given the specified system, an initial state, times at which solutions are desired, and values for the parameters, the function would instantiate the solution array with states at all K time points.  In generla, the `ode_solve` function would be highly polymorphic, requiring perhaps multiple arguments for initial states, multiple parameters, and multiple arguments for the returned solutions.  Arguments would also be required for data.  

This approach is perhaps the cleanest way to specify the system.  But the call to the solver would require all of the arguments in the appropriate order.  It would also be tricky to implement because of the polymorphism of the solver function and the need to convert the specification into a function that could be passed to the solver.

### Proposal 2: BUGS-like Function Syntax (Version A)

PKBugs and OpenBUGS take a very different approach, requiring the system to be defined as a function with a fixed signature.  

Version A would explicitly take in vectors for the state, parameters, data, and time:

```
vector[] harmonic_oscillator(vector[] state, 
                             vector[] parameters, 
                             vector[] data, 
                             real t) {
  vector[2] d_state;
  d_state[1] <- state[2];
  d_state[2] <- -state[1] - parameters[1] * state[2];
  return d_state;
}
```

The solver could then be called with a fixed signature 

```
vector[0] data;           // no constants involved in this system
vector[1] parameters;     // one parameter
vector[2] initial_state;  // 2D initial state
vector[K] solution_times; // K solution times
vector[2] solutions[K];   // K-array of 2D states for solutions
...
solutions <- ode_solve(harmonic_oscillator, 
                       data,
                       parameters,
                       intial_state,
                       solution_times);
```
    
### Proposal 3: BUGS-like Function Syntax (Version B)

BUGS's function syntax allows nodes (the equivalent of Stan variables) from the graph specification to be used in the function definition without being bound.  

So another way to define the system would be to not declare parameters or vectors, but to just use this:

```
vector[] harmonic_oscillator(vector[] x, real t) {
  vector[2] d_x;
  d_x[1] <- x[2];
  d_x[2] <- -x[1] - g * x[2];
  return d_x;
}
```

The same would be done for data. Calling the function could then skip the data and parameters, as in

```
vector[2] initial_state;  // 2D initial state
vector[K] solution_times; // K solution times
vector[2] solutions[K];   // K-array of 2D states for solutions
...
solutions <- ode_solve(harmonic_oscillator, 
                       intial_state,
                       solution_times);
```

Each of these variables, `initial_state`, `solution_times`, and `solutions`, would be a local variable, where `g` would be bound to a variable declared in the data or parameters block.

This is in some way the most natural approach from a user perspective because it does not require wrangling data and parameters out of vectors in the system definition or into vectors in the function call.  

The downside is that it'll be trickier to implement than proposal 2.


## Defining Densities with ODEs

Ben points out the following paper, which defines a density with an ODE:

* http://web.udl.es/Biomath/Group/Treballs/2006CSDA.pdf


## Pharmacokinetics and Toxicology Applications

In pharmacokinetic models, it's typical to have various external effects (thanks to Sebastian Weber for pointing these out and providing the text for the infusing):

* Dosing:  discontinous inputs to the differential equation in the form of "dosing".  For example, a drug is injected into a tissue, and complete diffusion is assumed, so it looks like a discontinuous jump.  

* Setting: Another typical use case is to set a compartment's concentration to a fixed value rather than increment with a delta function.  (There may also be other operations --- NONMEM seems pretty flexible in how it handles this kind of thing.)

* Infusing:  A third typical use is an infusion.  This means that one starts the infusion at a certain time point and stops it at a later time-point. In between drug is administered at a constant rate into one compartement. So instead of adding a delta peak to a compartement, one needs to increase the amount of drug in a compartement linearly with time. 

* Dosing Convenience Methods: steady-state dosing (i.e. dosing at a fixed time interval) and multiple dosing specifications (i.e. one only states "I want 50 dosing events with a between-dose time of 8h and a certain amount"). Both situations frequently arise in PK modeling and can be handled analytically very efficiently.

The only robust solution to dosing is to integrate between dosings.  This would, of course, require a more complicated input, with dose times and a delta for the state for each dose time.  (And the usual interpretation seems to be that if there is a dose and measurement at the same reported time, the measurement comes first.)

The question then arises as to whether we build dosing into the diff-eq model specifically in the form of state delta functions at specified times, or whether we force users to write in Stan.  We're leaning toward supporting dosing, but want to always allow users to bypass it and write everything in Stan itself.

We have some basic simulated PK/PD data that we've managed to fit using dosing.  Still in non-stiff setting, and so far we've only evaluated the PD model with the slower auto-diff the integrator approach.

## References

I've found these very useful so far.

1.  Adrian Sandu.  1997.  <a href="http://people.cs.vt.edu/asandu/Deposit/Sandu_ms_thesis.pdf">Sensitivity analysis of ODE via automatic differentiation</a>.  Department of Computer Science, University of Iowa.  (This one contains examples and code hints and a more gradual overview of the options.)

2.  Eberhard and Bischof. 1999.  <a href="http://www.ams.org/journals/mcom/1999-68-226/S0025-5718-99-01027-3/">Automatic differentiation of numerical integration algorithms</a>.  <i>Mathematics of computation</i>.  (This one contains the mathematical details defined most cleanly.)

3.  Frederic Bois.  <a href="http://www.gnu.org/software/mcsim/">MCSim</a>.  GNU Software.  (MCSim uses Metropolis, but otherwise does pretty much what we're looking for.  Frederic works with Andrew on pharmacokinetic models and is also helping on this Stan feature --- we wouldn't have gotten this far without him.)

4.  Lunn et al.  <i>The BUGS Book</i>.

And I believe this collection of models will be useful:

5.  https://code.google.com/p/bugsmodellibrary/

And here's a reference from Sebastian Weber about models used in Pharma:

6. http://www.lixoft.eu/wp-content/resources/docs/PKPDlibrary.pdf

We'd love to hear about other library collections. 