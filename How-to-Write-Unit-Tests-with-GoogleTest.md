#### What to Test

1. We're trying to make sure all the boundary conditions get called

    * zero-size inputs for containers
    * 0, NaN, inf, and -inf inputs for floating point

2. We want to make sure all tests get tickled and the appropriate exception types are thrown and documented.  

3.  We want to test that autodiff works

    * reverse-mode
    * forward mode: double (1st order), var (2nd order), fvar<var> (3rd order)

There should be a finite difference testing framework soon.  An alternative is to define a second, simpler templated versio of a function and test against that.

#### Using GoogleTest

The place to start for doc for GoogleTest (other than looking at our existing examples) are:

* [Primer](https://code.google.com/p/googletest/wiki/Primer): an overview

* [Advance Guide](https://code.google.com/p/googletest/wiki/AdvancedGuide): good description of available tests and best API guide I know
