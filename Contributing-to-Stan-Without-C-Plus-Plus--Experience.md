Because Stan is built upon a foundation of C++, contributions to the 
core of the project require extensive knowledge of C++ in addition to 
whatever statistical or algorithmic background might be needed for 
the given task.  This includes major additions such as new algorithms 
or new distributions exposed to the Stan modeling language.

On the other hand, all of that C++ is hidden from the user experience
by the interfaces.  Consequently there are countless of ways of 
improving the Stan user experience without having any familiarity
with C++ or professional software development.  Here we mention
a few possibilities for contributing to Stan without extensive C++
experience.

# Translating Course Materials and Exercises

With the growing popularity of Bayesian inference more and more
textbooks and courses are covering Bayesian methods.  This is
especially useful when the material focuses on a specific applied 
field so that the examples and exercises have direct relevance
to users.

Unfortunately many such sources build their material around 
software packages such as BUGS which limits their utility to
the Stan community.  Consequently an extremely useful contribution
is the translation of these course materials into Stan, first with
direct translations and then with updated model implementations
taking into account our current recommendations, especially for
priors.  

Some examples include the [translation by Hiroki Ito](https://github.com/stan-dev/example-models/tree/master/BPA) 
of the ecological models introduced in "Bayesian Population Analysis 
using WinBUGS --- A Hierarchical Perspective" (2012) by Marc Kéry and 
Michael Schaub, and the [translation by Martin Smira](https://github.com/stan-dev/example-models/tree/master/Bayesian_Cognitive_Modeling)
of the cognitive models introduced in “Bayesian Cognitive Modeling: 
A Practical Course” (2014) by Michael Lee and Eric-Jan Wagenmakers .

These translations are particularly amenable to those with domain
expertise in a given field who would like to learn about Stan.

# Translating Documentation From One Interface to Another

Much of our documentation, especially our case studies, is focused
on a particular interface.  While the interfaces are sufficiently
similar that much of the content immediately translates, there are
enough differences that the process can be frustrating.  

Consequently an immediate contribution is translating documentation
written for one interface into another, for example a case study
written in RStan to one written in PyStan or CmdStan.

# Localizing Documentation

One of the most exciting products of the growth of Stan has been
its adoption in international communities.  Unfortunately that
adoption can be limited by our documentation being available
almost exclusively in English.

Translations of Stan documentation, including the manual and
case studies, into other languages is a significant contribution
towards the scalability of the entire Stan project.  

A prominent example is the translation of much of the Stan
documentation by the Japanese Stan community, including the
[translation of the RStan installation instructions](https://github.com/stan-dev/rstan/wiki/RStan-Getting-Started-(Japanese)) by Kentaro Matsuura.

Another aspect of these efforts would be captioning the many
videos that are freely available on YouTube to improve accessibility
to hearing-impaired communities.

# Writing New Case Studies

Of course you shouldn’t be limited to just translating documentation!
If you have successfully applied Stan to an applied field then you 
are very welcome to submit a case study demonstrating that application.
These case studies can be focused on introducing the Stan community to 
your field, your field to Bayesian inference and Stan, or both!

# Improving Interface Support

A Bayesian analysis isn’t complete until the fit results have been
examined and reduced to interpretable products, especially visualizations.
Consequently there are many contributions to be made at the interface
level to improve this workflow that require only knowledge of the given
environment, such as R or Python.

A recent example is [BayesPlot](https://cran.r-project.org/web/packages/bayesplot/index.html)
which greatly expands the visualization facilities in RStan using 
ggplot2.  Similar functionality in Python would be particularly welcome.

# Improving Model Development Support

A key part of the Stan user experience is developing model specifications
with Stan programs, and this development is greatly enhanced with developer 
tools such as program validation and syntax highlighting.

Syntax highlighting, for example, has been implemented in 
[Atom](https://atom.io/packages/language-stan), 
[Emacs](https://github.com/stan-dev/stan-mode), and 
[Sublime](https://github.com/dougalsutherland/sublime-stan).  

With the diversity of integrated development environments, however, there 
are many more such contributions that can be made.

# Identifying Documentation Gaps

Most of the Stan documentation is written by the Stan team, and we have
been using Stan for so long that it becomes very easy to miss even critical
steps in the documentation.  For example, we may be missing important
pedagogical details in the Stan manual or case studies, or subtle details
in installation instructions.

Consequently a seemingly small but still important contribution to Stan
is keeping track of confusing or misleading documentation or gaps in our
presentations and letting us know through [Discourse](http://discourse.mc-stan.org).

# Documenting Performance Comparisons 

The rise of Bayesian inference has also given rise to additional
software package implementing dynamic Hamiltonian Monte Carlo,
like Stan, or other approximate computational algorithms.  While
the development team is continuously investigating new developments 
for inclusion into Stan, it is always helpful to have rigorous
performance comparisons not only for our own understanding but also
for the greater community that may not be sure which package is
more useful for their problems.

Consequently we welcome carefully done performance comparisons
between Stan and other software packages.  These should include
the model specified in both languages as well as a mathematical
description and then accuracy and speed comparisons with 
extensively documented hardware and software environments to
ensure reproducibility.

# Translating Analysis and Graphics Packages

Packages like `loo` and `bayesplot` in R work with RStan, but do not have counterparts in Python to work with PyStan.  Or in other interfaces.

At an even more ambitious level, there are not analogues of rstanarm or brms in Python.  

# Mathematical Support

Many of the contributions to the Stan language are limited not
just by development challenges but also by mathematical challenges.
This is particularly true for the introduction of new probability
density functions.

Each density function in Stan would ideally include the analytic
specification of its density, gradient, cumulative distribution
function, and random number generation, each of which requires 
significant research before any code is written.  Implementations
of any of these functions that rely on expansions require 
particular care to ensure sufficient numerical accuracy and
speed.  Indeed there are many such functions in Stan that could
use substantial improvements in this regard!

Documenting these functions and developing robust implementations
is a contribution naturally suited to those with strong mathematical
backgrounds and limited software experience.  It also serves as a 
convenient stepping stone for subsequent development of the
implementations in C++.
