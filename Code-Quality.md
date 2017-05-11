We want to institute some new policies on code, documentation, and testing quality. We are now (May 11 2017) okay with having two separate namespaces, one for the `stan::math` public API and another one inside called `internal` that will have hidden implementation details that likely only developers will ever notice. The `internal` code has similar quality requirements but doesn't have the same completeness requirements for docs and testing that API functions have. 

1.  Every `stan::math` function is part of our API and needs doxygen documentation indicating
    1. what its arguments are and whether they are input or output or both (`@param[in]`, `@param[out]`, or `@param[in,out]` tags)
    2.  what it's template parameters are (`@tparam` tag)
    3.  any constraints on the values of the arguments and an indication of what happens (typically exceptions) if the constraints are violated
    4.  which exceptions are thrown 
    5.  what the return value is (`@return` tag)

2.  Reused code needs to be broken out into reusable functions.

3.  Every `stan::math` function needs tests evaluating all the of constraints and conditions indicated by (1) as well as indicating that the right value is being computed

4.  Variables should have informative names (within reason).

5.  Keep dependencies clean.  In particular, we want to avoid
    1.  anything can depend on traits (but it should only depend on double traits if you use no agrad)
    2.  mcmc can depend on anything other than io (?)
    3.  memory shouldn't depend on anything (?)
    4.  `io` should only depend on the data types it needs and test functions (matrix and math tests only?)
    5.  `lang` shouldn't depend on anything else

6.  Absolutely minimal includes (include what you use).

8.  Tests should be parallel in paths to the source files (`src/stan/**` and `test/unit/**`).