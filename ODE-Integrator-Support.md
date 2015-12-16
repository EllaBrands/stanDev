[note:  Stan now has ODE support, but I'm leaving this page here for the references at the bottom; I've removed most of the outdated commentary]

## Defining Densities with ODEs

Ben points out the following paper, which defines a density with an ODE:

* http://web.udl.es/Biomath/Group/Treballs/2006CSDA.pdf

The first example is the Pearson family, which is defined by the diff eq

```
dy/dx = y * (mu - x) / (a + b * x + c * x^2) 
```

where `y` is the density, `x` is the variate, and `mu, a, b, c` are parameters.  This one has an explicit solution, but section 3.1 describes an algorithm for the GS distribution which doesn't have an explicit solution. 

## Systems Dynamics

Bill Harris sends these notes about systems dynamics models to stan-dev:

* http://www.metasd.com/models/ 

is a collection of system dynamics models.  Some of the Vensim models are just text, so you can examine those.  Others may require the Vensim model reader from 

* http://www.vensim.com/ 

or the iThink model player from 

* http://www.iseesystems.com/, 

both of which are free.  Follow the direction on Tom Fiddaman's page.

A system dynamics simulator pretty much requires the ability to specify a tabular nonlinear function with some sort of interpolation between defined points.  I know Stan is not compatible with the GPL, but I use GSL Akima interpolation routines in MCSim.  I'm confident Boost has something similar.

Finally, you might be interested in XMILE 

* http://xmile.systemdynamics.org/ 

and the XML Interchange Language for System Dynamics

* https://www.oasis-open.org/committees/tc_home.php?wg_abbrev=xmile

If you're not sure what system dynamics is, see 

* http://www.systemdynamics.org/what-is-s/ 

Despite them using DE notation, too, most system dynamics models are written as integral equations, something like 

```
state = integrate(netflow, initialconditions)
```


## Differential Algebraic Equations (DAE)

From Linas Mockus on stan-users:  

Request to handle DAEs because of their popularity in pharmacokinetic models:

 http://en.wikipedia.org/wiki/Differential_algebraic_equation

I think these are related to what Ben was calling "implicit definitions".

Linas also suggested looking at orthogonal collocation as a solution technique:

  http://en.wikipedia.org/wiki/Orthogonal_collocation

## Pharmacokinetics and Toxicology Applications

In pharmacokinetic models, it's typical to have various external effects (thanks to Sebastian Weber for pointing these out and providing the text for the infusing):

* Dosing:  discontinous inputs to the differential equation in the form of "dosing".  For example, a drug is injected into a tissue, and complete diffusion is assumed, so it looks like a discontinuous jump.  

* Setting: Another typical use case is to set a compartment's concentration to a fixed value rather than increment with a delta function.  (There may also be other operations --- NONMEM seems pretty flexible in how it handles this kind of thing.)

* Infusing:  A third typical use is an infusion.  This means that one starts the infusion at a certain time point and stops it at a later time-point. In between drug is administered at a constant rate into one compartement. So instead of adding a delta peak to a compartement, one needs to increase the amount of drug in a compartement linearly with time. 

* Dosing Convenience Methods: steady-state dosing (i.e. dosing at a fixed time interval) and multiple dosing specifications (i.e. one only states "I want 50 dosing events with a between-dose time of 8h and a certain amount"). Both situations frequently arise in PK modeling and can be handled analytically very efficiently.

The only robust solution to dosing is to integrate between dosings.  This would, of course, require a more complicated input, with dose times and a delta for the state for each dose time.  (And the usual interpretation seems to be that if there is a dose and measurement at the same reported time, the measurement comes first.)

The question then arises as to whether we build dosing into the diff-eq model specifically in the form of state delta functions at specified times, or whether we force users to write in Stan.  We're leaning toward supporting dosing, but want to always allow users to bypass it and write everything in Stan itself.

We have some basic simulated PK/PD data that we've managed to fit using dosing.  Still in non-stiff setting, and so far we've only evaluated the PD model with the slower auto-diff the integrator approach.

## Other Systems

* [Monolix](http://tutorial.lixoft.net)

* [NONMEM](http://www.iconplc.com/nonmem)

* [GNU MCSim](http://www.gnu.org/software/mcsim/)

## References

I've found these very useful so far.

1.  Adrian Sandu.  1997.  <a href="http://people.cs.vt.edu/asandu/Deposit/Sandu_ms_thesis.pdf">Sensitivity analysis of ODE via automatic differentiation</a>.  Department of Computer Science, University of Iowa.  (This one contains examples and code hints and a more gradual overview of the options.)

2.  Eberhard and Bischof. 1999.  <a href="http://www.ams.org/journals/mcom/1999-68-226/S0025-5718-99-01027-3/">Automatic differentiation of numerical integration algorithms</a>.  <i>Mathematics of computation</i>.  (This one contains the mathematical details defined most cleanly.)

3.  Frederic Bois.  <a href="http://www.gnu.org/software/mcsim/">MCSim</a>.  GNU Software.  (MCSim uses Metropolis, but otherwise does pretty much what we're looking for.  Frederic works with Andrew on pharmacokinetic models and is also helping on this Stan feature --- we wouldn't have gotten this far without him.)

4.  Lunn et al.  <i>The BUGS Book</i>.

### Collections of models

There are several collections of models that are useful.  Here are the BUGS models:

5.  https://code.google.com/p/bugsmodellibrary/

And here's a reference from Sebastian Weber about models used in Pharma:

6. http://www.lixoft.eu/wp-content/resources/docs/PKPDlibrary.pdf
