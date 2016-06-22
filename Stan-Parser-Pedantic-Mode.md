Background:  We see lots of errors in Stan code that could be caught automatically.  These are not syntax errors; rather, they're errors of programming or statistical practice that we would like to flag.  The plan is to have a "pedantic" mode of the Stan parser that will catch these problems (or potential problems) and emit warnings.  I'm imagining that "pedantic" will ultimately be the default setting, but it should also be possible for users to turn off pedantic mode if they really want.

Now I'll list a bunch of patterns that we'd like to catch.

To start with, I'll list the patterns in no particular order.  Ultimately we'll probably want to categorize or organize them a bit.

- Uniform distributions.  All uses of uniform() should be flagged.  Just about all the examples we've ever seen are either superfluous (as they just add a constant to the log density) or mistaken (in that they should have been entered as bounds on parameters).

- Parameter bounds of the form "lower=A, upper=B" should be flagged in all cases except A=0, B=1 and A=-1, B=1.

- Any parameter whose name begins with "sigma" should have "lower=0" in its declaration; otherwise flag.

- gamma(A,B) or inv_gamma(A,B) should be flagged if A = B < 1.  (The point is to catch those well-intentioned but poorly-performing attempts at improper priors.)

- if/else statements in the transformed parameters or model blocks:  These can cause problems with HMC so should probably be flagged as such.

- If code has no indentation, we should spit out a flag pointing users to our indentation conventions.

- If there are other common and easily-identifiable Stan programming errors, we should aim to catch them too.

Comments from Bob:

Most of these are going to have to be built into the parser
code itself to do it right, especially given the creative
syntax we see from users.

I don't know if the "sigma..." thing is worth doing.  Not
many of our users use "sigma" or "sigma_foo" the way Andrew does.

Do we also flag normal(0, 1000) and other attempts at very
diffuse priors?

The problem with a lot of this is that we'll have to be
doing simple blind matching.  Most of our users don't
use "sigma" (or "tau" the way Andrew does).  Nor do they
follow the sigma_foo standard---it's all over the place.

We can pick up literal values (and by literal, I mean
it's a number in the code).  We won't be able to pick up
more complex things like gamma(a, b) where a < 1 and b < 1,
because that's a run-time issue, not a compile-time issue,
for instance if a and b are data.

Should we also flag "lower=A" if A != 0 or A!= -1?   (Again,
we'll be able to pick up literal numbers there, not more
complex violations).

Conditionals (if-else) are only problematic if the conditions
depend on parameters.

Everyone wants to do more complicated things, like make sure
that if a variable is given a distribution with a bound that the
bound's declared in the parameters.  So no:

 real alpha;

 alpha ~ lognormal(...)

because then alpha isn't well defined. These things are going to
be a lot of work to track down and we'll again only be able to pick
out direct cases like this.

Comment from Michael:

I’m thinking of a -pedantic flag to the parser, which we can then call through the interfaces in various ways,
so it would make sense to built everything up in the parser itself.

Bob replies:

The patterns in question are complex syntactic relationships in some clases, like the link between a variable's declaration and use. If we start cluttering up the parser trying to do all this on the fly, things could get very ugly (in the code) very fast, as the underlying parser's just a huge pain to work with.

Mike:

So what’s the best place to introduce these checks? It’s not dissimilar for what’s already done with the Jacobian checks, no?

Bob:

It depends what they are.  I should've been clearer that I meant on the fly as in find the issue as it's parsing.

There are two approaches --- plumb enough information top down through the parser so that we can check inside of a single expression or statement --- that's what the Jacobian check does now.  To do that, the source of every variable needs to be plumbed through the entire parse tree.  If we do more of that it's going to get ugly.  Some of these are "peephole" warnings --- things you can catch just locally.

For others, we have the AST and can design walkers for it that walk over the code any old way and report.

Or, we could do something hacky with regexes along the lines of cpplint (a Python program, not something plumbed into the C++ parser, at least as far as I know).