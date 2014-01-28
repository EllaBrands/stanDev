## Overview

We are going to add support for ordinary differential equation (ODE) solvers (i.e., integrators) to Stan.  The language will be extended to allow ODEs to be defined in Stan and then used for solution.  We want to support both easy and stiff ODEs and would like to provide error control on both the ODE solutions and their gradients with respect to system parameters.

## Working Code Branch

This is going to be a rather big feature, so I hope an issue will work to consolidate discussion.  There's a branch Daniel and I started for this project:

```
feature/ode_experiment
```

It currently contains a fully working example of Stan fitting a model using an ode.  The ode is a simple harmonic oscillator.



We built a special function in Stan using Boost's odeint package.  We use auto-diff on the integrator (with error control and interpolation).   And it works for simulated data (R package to do simulation included). 

Next, we're going to try the alternative method of autodiff-ing the system of differential equations (manually, to start), to give us error control on the gradients and make sure they don't suffer any unexpected behavior from autodiff-ing through the integrator (I still don't understand why adaptation should be a problem).

The simple auto-diff of the integrator approach is in:

```
src/stan/math/functions/ho.hpp
```

The integration of the coupled system including the derivatives is in:

```
src/stan/math/functions/ho2.hpp
```

The latter approach, `ho2.hpp`, is about 5 times as efficient, and also provides error control.  The issue we have now is that we can't define this coupled system by reverse-mode auto-diff because we only have a single auto-diff stack.

## Example: Harmonic Oscillator

This is based on the <a href="http://www.boost.org/doc/libs/1_53_0/libs/numeric/odeint/doc/html/boost_numeric_odeint/tutorial/harmonic_oscillator.html">harmonic oscillator example</a> from <a href="http://www.boost.org/doc/libs/1_53_0/libs/numeric/odeint/doc/html/index.html">Boost's odeint package</a>.

### Defining the System of Equations

#### Proposal 1

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

The ```dynamics``` block would allow arbitrary code.  There would also need to be a data block to define constants.  

PKBugs and OpenBUGS take a very different approach, requiring the system to be defined as a function with signature:

```
vector[] harmonic_oscillator(vector[] state);
```

and taking the other parameters to be implicit.  Then there's an explicit call to a function `integrate` that takes the system name, an initial state and sequence of times, and returns a vector of solutions at those times:  

```
vals <- integrate(harmonic_oscillator, initial_state, eval_times);

We've also been thinking of adding functions to Stan, which would define the above as

```
vector[] harmonic_oscillator(vector[] x) {
  vector[2] d_x;
  d_x[1] <- x[2];
  d_x[2] <- -x[1] - gamma * x[2];
  return d_x;
}
```

Note that ```gamma``` is not bound in the function scope.  We haven't sorted through scoping options for functions yet, but I (Bob) don't like this kind of R-like function design --- I'd much rather see ```gamma``` passed in as an argument.                  

### Calling the Integrator in Stan

This would then be available as a function to be called in a Stan model, such as our working example:

```
data {
  int<lower=0> N;
  int<lower=1> M;
  real t[N];
  matrix[N,M] x_obs;
  vector[M] x0;
}
parameters {
  real<lower=-10,upper=10> gamma;
  real<lower=0, upper=1> sigma[2];
}
model {
  matrix[N,M] x_hat;
  x_hat <- harmonic_oscillator(t, x0, gamma);
  for (n in 1:N)
    x_obs[n] ~ normal(x_hat[n], sigma);
}
```

The basic idea is that the ODE defines a function ```harmonic_oscillator``` that takes an array of times for which solutions to the ODE should be returned.  Note that the ODE function returns ```x_hat```, which is an array of values, where the values in the array are of the same type as the state and the array has the same size (number of entries) as the time argument ```t```.  The value ```x0``` is the initial state for the ODE.  The argument gamma is for the parameter to the ode;  in general, there may be more than one parameter.  Note also that the parameter ```gamma``` here is also a parameter to the Stan model, meaning it's something we need to differentiate with respect to if we are going to run Hamiltonian Monte Carlo samplers or gradient-based optimizers or variational Bayes posterior approximators.

#### More General States?

I don't see how to allow the state to be anything other than a single value because Stan doesn't have functions that return sequences.  What I'd like to be able to do in Stan is this:

```
state {
   real x;
   real y;
}
```

and then

```
(x_hat,y_hat) <- ode_fun(t,x0,y0,...);
```

It's been in the back of my mind to add something like this to Stan's assignment language, but it'll take some more design and then probably a fair bit of coding.

## Methods for Gradients

All of our applications require integrators with error control and interpolation (like the Dopri integrator from Boost).  To handle stiff systems, we will need an integrator that handles Jacobians (like the Rosenbrock integrator from Boost).

There are four basic approaches:

1.  auto-diff the integrator  (+gradient error control, -stiff)

2.  auto-diff the gradient, then integrate (+gradient error control, -stiff)

3.  auto-diff system Jacobian, then auto-diff the stiff integrator (-gradient error control, +stiff)

4.  auto-diff the gradient, then auto-diff the coupled system's Jacobian, then integrate (+gradient error control, +stiff).

We have 1. working for the Harmonic oscillator model shown above already.

## Dosing

In pharmacokinetic models, it's typical to have various external effects (thanks to Sebastian Weber for pointing these out and providing the text for the infusing):

* Dosing:  discontinous inputs to the differential equation in the form of "dosing".  For example, a drug is injected into a tissue, and complete diffusion is assumed, so it looks like a discontinuous jump.  

* Setting: Another typical use case is to set a compartment's concentration to a fixed value rather than increment with a delta function.  (There may also be other operations --- NONMEM seems pretty flexible in how it handles this kind of thing.)

* Infusing:  A third typical use is an infusion.  This means that one starts the infusion at a certain time point and stops it at a later time-point. In between drug is administered at a constant rate into one compartement. So instead of adding a delta peak to a compartement, one needs to increase the amount of drug in a compartement linearly with time. 
</blockquote>

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

We'd love to hear about other library collections.