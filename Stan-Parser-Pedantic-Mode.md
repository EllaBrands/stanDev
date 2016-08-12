Background:

We see lots of errors in Stan code that could be caught automatically.  These are not syntax errors; rather, they're errors of programming or statistical practice that we would like to flag.  The plan is to have a "pedantic" mode of the Stan parser that will catch these problems (or potential problems) and emit warnings.  I'm imagining that "pedantic" will ultimately be the default setting, but it should also be possible for users to turn off pedantic mode if they really want.


Choices in implementation:

- It would be fine with me to have this mode always on, but I could see how some users would be irritated if, for example, they really want to use an inverse-gamma model or if they have a parameter called sigma that they want to allow to be negative, etc.  If it is _not_ always on, i'd prefer it to be on by default, so that the user would have to declare "--no-check-mode" to get it to _not_ happen.

- What do others think?  Would it be ok for everyone to have it always on?  If that's easier to implement, we could start with always on and see how it goes, then only bother making it optional if it bothers people.

- I'm thinking these should give Warning messages.  Below I'll give a suggestion for each pattern.

- And now I'm thinking we should have a document (not the same as this wiki) which explains _why_ for each of our recommendations.  Then each warning can be a brief note with a link to the document.  I'm not quite sure what to call this document, maybe something like "Some common problems with Stan programs"?  We could subdivide this further, into Problems with Bayesian models and Problems with Stan code.

Now I'll list a bunch of patterns that we'd like to catch.

To start with, I'll list the patterns in no particular order.  Ultimately we'll probably want to categorize or organize them a bit.

- Uniform distributions.  All uses of uniform() should be flagged.  Just about all the examples we've ever seen are either superfluous (as they just add a constant to the log density) or mistaken (in that they should have been entered as bounds on parameters).

Warning message:  "Warning:  On line ***, your Stan program has a uniform distribution.  The uniform distribution is not recommended, for two reasons:  (a) Except when there are logical or physical constraints, it is very unusual for you to be sure that a parameter will fall inside a specified range, and (b) The infinite gradient induced by a uniform density can cause difficulties for Stan's sampling algorithm.  As a consequence, we recommend soft constraints rather than hard constraints; for example, instead of giving an elasticity parameter a uniform(0,1) distribution, try normal(0.5,0.5)."

- Parameter bounds of the form "lower=A, upper=B" should be flagged in all cases except A=0, B=1 and A=-1, B=1.

Warning message:  "Warning:  On line ***, your Stan program has a parameter with hard constraints in its declaration.  Hard constraints are not recommended, for two reasons:  (a) Except when there are logical or physical constraints, it is very unusual for you to be sure that a parameter will fall inside a specified range, and (b) The infinite gradient induced by a hard constraint can cause difficulties for Stan's sampling algorithm.  As a consequence, we recommend soft constraints rather than hard constraints; for example, instead of constraining an elasticity parameter to fall between 0, and 1, leave it unconstrained and give it a normal(0.5,0.5) prior distribution."

- Any parameter whose name begins with "sigma" should have "lower=0" in its declaration; otherwise flag.

Warning message:  "Warning:  On line ***, your Stan program has a unconstrained parameter with a name beginning with "sigma".  Parameters with this name are typically scale parameters and constrained to be positive.  If this parameter is indeed a scale (or standard deviation or variance) parameter, add lower=0 to its declaration."

- Parameters with positive-constrained distribution (such as gamma or lognormal) and no corresponding constraint in the definition.

Warning message:  "Warning:  Parameter is given a constrained distribution on line *** but was declared with no constraints, or incompatible constraints, on line ***.  Either change the distribution or change the constraints."

- gamma(A,B) or inv_gamma(A,B) should be flagged if A = B < 1.  (The point is to catch those well-intentioned but poorly-performing attempts at improper priors.)

Warning message:  "Warning:  On line ***, your Stan program has a gammma, or inverse-gamma, model with parameters that are equal to each other and set to values less than 1.  This is mathematically acceptable and can make sense in some problems, but typically we see this model used as an attempt to assign a noninformative prior distribution.  In fact, priors such as inverse-gamma(.001,.001) can be very strong, as explained by Gelman (2006).  Instead we recommend something like a normal(0,1) or student_t(4,0,1), with parameter constrained to be positive."

- if/else statements in the transformed parameters or model blocks:  These can cause problems with HMC so should probably be flagged as such.

Warning message:  Hmmm, I'm not sure about this one!

- Code has no indentation.

Warning message:  "Warning:  Your Stan code has no indentation and this makes it more difficult to read.  See *** for guidelines on writing easy-to-read Stan programs."

- Code has blank lines.

Warning message:  "Warning:  Your Stan code has blank lines and this makes it more difficult to read.  See *** for guidelines on writing easy-to-read Stan programs."

- Undefined variables.  Bob describes this as a biggy that's a lot harder to code.

Warning message:  "Warning:  Variable ** is used on line *** but is nowhere defined.  Check your spelling or add the appropriate declaration."

- Variables defined lower in the program than the variable was used.

Warning message:  "Warning:  Variable ** is used on line *** but is not defined until line ****.  Declare the variable before it is used."

- Large or small numbers.

Warning message:  "Try to make all your parameters scale free.  You have a constant in your program that is less than 0.1 or more than 10 in absolute value on line **.  This suggests that you might have parameters in your model that have not been scaled to roughly order 1.  We suggest rescaling using a multiplier; see section *** of the manual for an example.

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