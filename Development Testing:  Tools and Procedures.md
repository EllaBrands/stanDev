> [note: Daniel is writing this and may not represent the Stan team until this is removed]

***

# Overview
1. Kinds of tests (testing levels).
2. How to invoke tests.
3. Technical details. (Google test; our distribution testing framework)
4. If providing a new feature, what's the minimum level of testing required?
5. The old text.

The [Stan developer process](https://github.com/stan-dev/stan/wiki/Developer-Process#new-and-altered-code-is-tested) specifies that all new and altered code should be tested.  We believe that good test coverage is necessary given the size and complexity of the Stan code base (or any code base consisting of more than a Perl 1-liner, and quite frankly, we don't trust that Perl 1-liner).   For more on how and why to test, see this [wikipedia article](http://en.wikipedia.org/wiki/Software_testing).

We know that there are many bugs in our code.  With good test coverage, when a bug is found, these tests allow us to demonstrate:
 - that the bug exists
 - that the bug has been fixed
 - that the fix has won't break something else
   
We can't prove that the fix hasn't introduced new, unknown bugs, but we can prove that this fix won't cause something else to break.

To show that the bug exists, write a test that tickles the bug.  Before going on to fix the bug, run the test, verify that it causes failure, and add and commit the test to the Stan repo.

Now fix the bug.  To verify that the bug has been fixed, re-run the test and verify that it no longer fails.

Verify that the fix hasn't broken existing code.  To do this, we re-run existing tests.  Before submitting a pull request, we expect the developer to run those tests which directly excercise the code.  At a minimum, if you fix a bug in `stan/foo/bar.hpp`, then you should run all tests for `src/stan/foo`.

When a pull request is submitted, the entire Stan test suite will be run on our [Continuous Integration](https://github.com/stan-dev/stan/wiki/Continuous-Integration) server.




***

## 1. Kinds of tests




### Unit tests

### Model tests

### Functional tests





## 2. How to invoke tests.

Stan test code live in the directory `src/test` in the Stan repository.

Stan currently uses the Unix `make` utility, specifically Gnu Make 3.8, to build and test Stan.  See [Testing Stan using Gnu Make and Python](https://github.com/stan-dev/stan/wiki/Testing-Stan-using-Gnu-Make-and-Python) for details.

We plan to migrate to the [Cmake metabuild system](https://github.com/stan-dev/stan/wiki/Building-Stan-with-CMake).



  


## 3. Technical details. (Google test; our distribution testing framework)

We'll try to describe some of the implementation details of the tests for Stan.

- Test naming convention
    
    All tests are named `*_test.cpp` and are located in subdirectories of `src/test`.
    
    The test executable for `src/test/*_test.cpp` will be created at `test/*` (add `.exe` for Windows). The rule is to strip the leading `src/` and the `_test.cpp` from the filename.
    
- How to run a specific test
    
    The make target is test executable. For example, if the test is located at `src/test/foo/bar_test.cpp`, the command to build the test is ```> make test/foo/bar```.
    
    We tooled the makefile to run the test on a build. We have a second target to force running of the test, even if it is built. 
    ```> make runtest/test/foo/bar```
    (add the prefix `runtest/`)
    

## 4. If providing a new feature, what's the minimum level of testing required?
## 5. The old text.

    
- How to add new tests
- How the autogenerated tests work
- What tests to add for a new file?
- To what extent should methods be tested?
- Examples

