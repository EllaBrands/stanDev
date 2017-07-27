Some things to do:

- Fast regression with point estimates and fixed priors (using rstanarm using the optimize=TRUE setting to replace bayesglm):  Currently rstanarm in optimize=TRUE setting is kinda slow because it does simulations and computes second derivatives.  I'm not quite clear on which of these things (or others) is the CPU hog, but we'll want some "light" versions that will run fast.

- GMO

- Other ideas, I'm sure.