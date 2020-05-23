## GoogleTest Unit Tests for C++

#### What to Test

1. We're trying to make sure all the boundary conditions get called

    * zero-size inputs for containers
    * 0, NaN, inf, and -inf inputs for floating point

2. We want to make sure all tests get tickled and the appropriate exception types are thrown and documented.  

3. We want to test that autodiff works

    * reverse-mode
    * forward mode: double (1st order), var (2nd order), fvar<var> (3rd order)

There should be a finite difference testing framework soon.  An alternative is to define a second, simpler templated versio of a function and test against that.

#### Using GoogleTest

The place to start for doc for GoogleTest (other than looking at our existing examples) are:

* [Primer](https://code.google.com/p/googletest/wiki/Primer): an overview

* [Advance Guide](https://code.google.com/p/googletest/wiki/AdvancedGuide): good description of available tests and best API guide I know

#### Common test types

The documentation (above) is excellent so check there for details and the complete list of tests.  The following is a small sample of commonly used test types in Stan and links to the relevant sections of the GoogleTest documentation.

* [EXPECT/ASSERT](https://code.google.com/p/googletest/wiki/Primer#Assertions) EXPECT records the result but allows further tests in the same function, ASSERT exits the current function without cleanup. Stan unit tests use both forms.
* [EXPECT_TRUE](https://code.google.com/p/googletest/wiki/Primer#Basic_Assertions) EXPECT_TRUE/FALSE test that the condition is TRUE/FALSE.
* [EXPECT_EQ](https://code.google.com/p/googletest/wiki/Primer#Binary_Comparison) Tests that a calculated value equals (==) an expected result exactly, not appropriate for floating point comparison.
* [EXPECT_FLOAT_EQ](https://code.google.com/p/googletest/wiki/AdvancedGuide#Floating-Point_Comparison) Tests that a floating point calculation results to known output values.
* [EXPECT_NEAR](https://code.google.com/p/googletest/wiki/AdvancedGuide#Floating-Point_Macros) Like EXPECT_FLOAT_EQ but a third argument sets the threshold used. 
* [ASSERT_THROW](https://code.google.com/p/googletest/wiki/AdvancedGuide#Exception_Assertions) Tests that a specific exception type is thrown.
* [ASSERT_NO_THROW](https://code.google.com/p/googletest/wiki/AdvancedGuide#Exception_Assertions) Tests that the statement does not throw. Some parts of the code are not supposed to throw... not sure which.

~                                                               