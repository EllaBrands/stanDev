Background:  We see lots of errors in Stan code that could be caught automatically.  These are not syntax errors; rather, they're errors of programming or statistical practice that we would like to flag.  The plan is to have a "pedantic" mode of the Stan parser that will catch these problems (or potential problems) and emit warnings.  I'm imagining that "pedantic" will ultimately be the default setting, but it should also be possible for users to turn off pedantic mode if they really want.

Now I'll list a bunch of patterns that we'd like to catch.

To start with, I'll list the patterns in no particular order.  Ultimately we'll probably want to categorize or organize them a bit.

- Uniform distributions.  All uses of uniform() should be flagged.  Just about all the examples we've ever seen are either superfluous (as they just add a constant to the log density) or mistaken (in that they should have been entered as bounds on parameters).

- Parameter bounds of the form "<lower=A, upper=B>" should be flagged in all cases except A=0, B=1.

- Any parameter whose name begins with "sigma" should have <lower=0> in its declaration; otherwise flag.

- gamma(A,B) or inv_gamma(A,B) should be flagged if A = B < 1.  (The point is to catch those well-intentioned but poorly-performing attempts at improper priors.)