Sometimes a new function is needed in Stan.  This function may compute some special mathematical function, or may be intended to speed up automatic differentiation by avoiding intermediate computations and directly computing the gradients.  This page documents the basic steps necessary to do this.

This document aims to describe what we'll need if we're going to make the function available as part of Stan.  If you only want a new function for your own uses, you only need step (1) below. 

Before doing anything, please read the Developer Process document, setup your git repository and create a `feature/<new function>` branch.  The steps below should, roughly speaking, consist of one commit each in this repository.

#### Implementing the Function

Implement the new function in the appropriate location and include it in the appropriate header files.The best way to figure out where this is and how to implement the function is to find a similar function already in the API and use it as a model.  You can grep to see where it's included.  

It is important to implement the function in such a way that it is templated such that it can be called with auto-diff variables for parameters and double values for data.  Again, see the existing functions for examples.

#### Expose Function Signature to Stan Models

Expose the function to the parser by adding the appropriate code to src/stan/gm/function_signatures.hpp and implement function signature tests by adding models to src/test/gm/model_specs/compiled/.

#### Add to Relevant Header File

Depending on where you put the function, you'll need to include its definition file in the appropriate header include to make sure it's visible to models.  For instance, if the function is going into `stan/math/functions/...` then it should be included into `stan/math/functions.hpp`

#### Unit Testing

Implement a unit test for the function which covers both the value (i.e., that it computes the right thing), the gradients of the function, and the error behavior (e.g., when the function is passed illegal values).  Looking at a unit test of a similar function will help give a sense of how to do this.  

You should check all the ways in which it can be called --- that is all permutations of scalars/vectors and double and autodiff variables.

An individual unit test, e.g., src/test/unit-agrad-rev/functions/log_test.cpp, can be run with `make test/unit-agrad-rev/functions/log`

After the individual test passes, make sure that `make test-unit` and `make test-headers` both pass.

Add a test with all signatures in the form of a model in `test/gm` to make sure it's compilable from within a Stan model.

#### Documentation

Add reference documentation to the manual.  Ideally with a formula for it.  Include a reference if it's uncommon.

#### Submit

Submit a pull request on GitHub.  We'll review the code, doc and tests, perhaps iterate over fixes, verify that it passes tests, and then merge it into Stan.  
