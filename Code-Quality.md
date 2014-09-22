We want to institute some new policies on code, documentation, and testing quality.

1.  Every single function needs doxygen documentation indicating
    a. what its arguments are and whether they are input or output or both (`@param[in]`, `@param[out]`, or `@param[in,out]` tags)
    b.  what it's template parameters are (`@tparam` tag)
    c.  any constraints on the values of the arguments and an indication of what happens (typically exceptions) if the constraints are violated
    d.  which exceptions are thrown 
    e.  what the return value is (`@return` tag)

2.  Reused code needs to be broken out into reusable functions.

3.  Every function needs tests evaluating all the of constraints and conditions indicated by (1) as well as indicating that the right value is being computed

4.  Variables should have informative names (within reason).

5.  Keep dependencies clean.  In particular, we want to avoid
    a.  non-matrix code depending on Eigen Matrix
    b.  non-agrad code depending on agrad (and non-matrix agrad code depending on matrix)
    c.  agrad rev or agrad fwd depending on each other (what about mixed tests -- I think they should be separated out?)
    d.  anything can depend on traits (but it should only depend on double traits if you use no agrad)
    e.  mcmc can depend on anything other than io (?)
    f.  memory shouldn't depend on anything (?)
    g.  io should only depend on the data types it needs and test functions (matrix and math tests only?)
    h.  gm sholdn't depend on anything else (and should be renamed)
    i.  meta should be split out based on its dependencie, but basic meta shouldn't depend on matrix or on agrad, so that's meta/**, meta/matrix/** should be parallel to agrad/rev/meta/** and agrad/fwd/meta/**.
    j.  optimization can depend on anything
    k.  prob should only depend on OperandsAndPartials and traits and the multivariate ones on matrix
    l.  command will go away (move to CmdStan)
    m.  common ???

6.  Absolutely minimal includes (include what you use).  That is, no bringing in all of Eigen/Dense just to use a matrix multiplication and no bringing in all of var.hpp just to use an autodiff variable.

7.  Keep development parallel (in terms of file names) in math, agrad/rev and agrad/fwd.

8.  Tests should be parallel in paths to the source files (src/stan/** and test/unit/**).