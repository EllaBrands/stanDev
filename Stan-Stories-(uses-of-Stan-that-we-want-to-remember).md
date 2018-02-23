This page collects examples of Stan use out in the world that will help us understand our impact. 

### rethinking

R package behind the *Statistical Rethinking* book we recommend.

Developed by Richard McElreath at University of Leipzig.


### ScalaStan 

Embedded version of domain-specific language that compiles down to Stan.

Developed by Joe Wingbermuehle at Cibo Technologies.

### Prophet 

https://research.fb.com/prophet-forecasting-at-scale/

Time-series forecasting with periodic (e.g., seasonal, weekly) and ad-hoc (e.g., pre- and post-holiday) effects.  Interfaces in R and Python.

Developed by Sean J. Taylor and Ben Letham at Facebook.  

### brms

https://cran.r-project.org/web/packages/brms/index.html

Widely used R package providing expression-based modeling.  It's been around longer than rstanarm.  

Developed by Paul-Christian Bürkner at University of Münster.

### torsten

R package developed for pharmacological modeling in RStan, with common data structures and built-in modeling.  

Develeloped by yonicd (GitHub handle), Charles Margossian, and Yi Zhang at Metrum Research, Inc.

### PMXStan

http://andrewgelman.com/2015/10/05/pmxstan-an-r-package-to-facilitate-bayesian-pkpd-modeling-with-stan/

R interface to Stan with pharmacology data structures and built-in models.

Developed by Yuan Xiong, David A. James, Fei He, and Wenping Wang at Novartis.


### RStudio downloads run 2/18 

`> install.packages("dlstats")`

`> library(dlstats)`

`> s <- cran_stats("rstan")`

`> tail(s)`

`35 2017-11-01 2017-11-30     11980   rstan`

`36 2017-12-01 2017-12-31     12294   rstan`

`37 2018-01-01 2018-01-31     16215   rstan`

`> sum(s$downloads)`

`[1] 250148`


So, 250K downloads **just through RStudio**.  brms has 100K,
rstanarm 65K, and prophet 40K.  rstanarm's at 3K/month.  That
doesn't include any of the other R mirrors, or any of the other
interfaces.

Our wild-ass guess is 20K to 100K users, but it could be more
if you count brms, rstanarm, etc.  Again, hard to say.  Regular
users?  People who learned it in a class?  Applied it once with
a colleague?

### Japanese Textbook

### Courses 
* There are dozens if not hundreds of courses being taught
using Stan for inference

### Publications
* There are hundreds of published papers using Stan
for inference.  Here's an abbreviated list from a year ago of topic areas:
  - Biological sciences: clinical drug trials, entomology, opthalmology,
      neurology, genomics, agriculture, botany, fisheries, cancer biology,
      epidemiology, population ecology, neurology, pharmacology
  - Physical sciences: astrophysics, molecular biology, oceanography,
      climatology, biogeochemistry
  - Social sciences: population dynamics, psycholinguistics, social networks,
      political science, surveys
  - Other: materials engineering, finance, actuarial, sports, public health,
      recommender systems, educational testing, equipment maintenance,
      fleet management

### Books
* There are also half a dozen books either wholly or partly involving
Stan (McElreath, Kruschke, Korner-Nievergelt et al., Gelman et al., etc.)
These are listed on the web site.  One just came out devoted to astrophysics.

### Jobs
Lots of jobs are showing up explicitly advertising for Stan
expertise, but it's usually in the context of "INLA, BUGS, JAGS, Stan,
or PyMC";  those are really our competition among statisticians (though
PyMC is relatively rare in this context because Python's not very popular
among applied statisticians).  Some examples from current listings:
"Expertise in statistical software, such as R, SAS, WinBUGS, STAN,
and/or Matlab" and "Absolutely high fluency with R and/or Python and
Bayesian MCMC tools such as JAGS or STAN".  The Yankees advertised on
our sites and other major league teams have advertised.  

### Projects

LIGO gravitational wave experiments. 

Neutrino experiments at MIT (Talia from StanCon's group)

### Other metrics

> 'mc-stan.org' has 1060 cites in google scholar

It's even higher than that if you look for other things like
the phrase "Stan Development Team".  I hunted deeper a year or
two ago and found over 100 papers using Stan to fit their models.
There's a lot that just cite us in the "JAGS and BUGS and Stan" sort
of way.   It's really hard to count.