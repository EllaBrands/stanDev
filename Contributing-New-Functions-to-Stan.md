Sometimes a new function is needed in Stan.  This function may compute some special mathematical function, or may be intended to speed up automatic differentiation by avoiding intermediate computations and directly computing the gradients.  This page documents the basic steps necessary to do this.

Before doing anything, please read the Developer Process document, setup your git repository and create a `feature/<new function>` branch.  The steps below should, roughly speaking, consist of one commit each in this repository.

1. Implement the new function in the appropriate location and include it in the appropriate header files.  The best way to figure out where this is and how to implement the function is to find a similar function already in the API and use it as a model.

2. Implement a unit test for the function which covers both the value (i.e., that it computes the right thing), the gradients of the function and the error behaviour (e.g., when the function is passed illegal values).  Looking at a unit test of a similar function will help give a sense of how to do this.  An individual unit test, e.g., src/test/agrad/rev/log_test.cpp, can be run with `make test/agrad/rev/log`

3. Expose the function to the parser by adding the appropriate code to src/stan/gm/function_signatures.hpp and implement function signature tests by adding models to src/test/gm/model_specs/compiled/.

4. Add reference documentation to the manual.

5. Ensure that `make test-unit` passes and submit a pull request on GitHub.

