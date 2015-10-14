Sometimes a new function is needed in Stan.  This function may compute some special mathematical function, or may be intended to speed up automatic differentiation by avoiding intermediate computations and directly computing the gradients.  This page documents the basic steps necessary to do this.

To be available as part of the Stan language, a function must be added in two places.  First, the C++ relevant code should be added to the [Stan math library](https://github.com/stan-dev/math), along with tests that demonstrate its intended behavior.  Second, the function must be added to the Stan language in the [Stan repository](https://github.com/stan-dev/stan), along with tests that demonstrate that it is parsed as intended.  If you only want a new function available through the Stan math library C++ API it does not need to be exposed to the Stan language. 

Before doing anything, please read the Developer Process document, setup your git repository and create a `feature/<new function>` branch.  The steps below should, roughly speaking, consist of one commit each in this repository.

#### Implementing the Function

Functions should be implemented in the GitHub repo stan-dev/math, not in this repo, stan-dev/stan.  That means you need a new issue for the math repo and unit tests there.  When that's all done and tested, you can come back to do the Stan pieces of the puzzle, which are straightforward.

Implement the new function in the appropriate location, most likely in the [stan/math](https://github.com/stan-dev/math) repository and include it in the appropriate header files.  The best way to figure out where this is and how to implement the function is to find a similar function already in the API and use it as a model.  You can grep to see where it's included.  

It is important to implement the function in such a way that it is templated such that it can be called with auto-diff variables for parameters and double values for data.  Again, see the existing functions for examples.

#### Expose Function Signature to Stan Models

Expose the function to the parser by adding the appropriate code to [`src/stan/lang/function_signatures.hpp`](https://github.com/stan-dev/stan/blob/develop/src/stan/lang/function_signatures.h) and implement function signature tests by adding models to `src/test/gm/model_specs/compiled/`.

#### Add to Relevant Header File

Depending on where you put the function, you'll need to include its definition file in the appropriate header include to make sure it's visible to models.  For instance, if the function is going into `stan/math/functions/...` then it should be included into `stan/math/functions.hpp`.

#### Unit Testing

Implement a unit test for the function which covers both the value (i.e., that it computes the right thing), the gradients of the function, and the error behavior (e.g., when the function is passed illegal values).  Looking at a unit test of a similar function will help give a sense of how to do this.  

You should check all the ways in which it can be called --- that is all permutations of scalars/vectors and double and autodiff variables.

An individual unit test, e.g., `src/test/unit-agrad-rev/functions/log_test.cpp`, can be run with `make test/unit-agrad-rev/functions/log`.

After the individual test passes, make sure that `make test-unit` and `make test-headers` both pass.

Add a test with all signatures in the form of a model in `test/gm` to make sure it's compilable from within a Stan model.

#### Model Tests

We need models in test_models instantiating all the possible ways the function can be instantiated.  Examples are in `src/test/test-models/good` for ones that pass and in `src/test/test-models/bad` for ones that don't.  Specifically, the function signature tests should go in `src/test/test-models/good/function-signatures`.  Then there are drivers for these tests in `src/test/unit/lang/parser` --- look at the ones that are there to see how to test that models don't parse or throw the right warnings when trying to parse them.

#### Documentation

Add reference documentation to the manual.  Ideally with a formula for it.  Include a reference if it's uncommon.

#### Submit

Submit a pull request on GitHub.  We'll review the code, doc and tests, perhaps iterate over fixes, verify that it passes tests, and then merge it into Stan.  
