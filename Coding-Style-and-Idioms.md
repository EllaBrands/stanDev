We generally follow the [Google Style Guide](https://google.github.io/styleguide/cppguide.html) except for the exceptions listed in the following sections. We use a few tools to automate style and formatting checking, but they don't completely capture our formatting and style guidelines.

## Supported Cpp Versions and Compilers

### Tested Toolchains and OSes

#### On PRs:
* mac / clang 6 / libc++ Threading
* linux / gcc 5 / stdlibc++ OpenMPI
* mac clang 6 libc++ OR linux clang 6 libc++ distribution tests (without row vectors)
* Windows gcc 4.9.3

#### Additional tests on merge to develop 
* linux / gcc 5 / stdlibc++ Threading unit tests
* mac / clang 6 libc++ OR linux / gcc 5 stdlibc++[1] GPU unit tests
* mac clang 6 libc++ OR linux clang 6 stdlibc++[2] distribution tests (with row vectors)

So there will be a little bit of non-determinism for the sake of running tests more quickly on available hardware.

[1] the GPU libraries are currently broken on Linux, so currently they run on Mac and Linux is an aspirational target.

[2] clang 6 is also aspirational on that Linux box - clang 3.8 is the only one that works without libc++ right now. 

### C++ Standard library version
If you're on Mac, this is probably libc++, and on Windows or Linux this is probably GNU's stdlibc++. We should support both for the versions of compilers we support.

### C++ Language Features
We are not yet testing the Math repo on GCC 4.9.3, but we will soon and we need that compiler to work to support the way RStan on Windows links with the R binary at runtime. This is our limiting compiler and means we can use:

#### [C++11 features](http://blog.smartbear.com/c-plus-plus/the-biggest-changes-in-c11-and-why-you-should-care/)
* everything except for Expression SFINAE

#### C++14 features
* auto and decltype(auto) return types
* generic lambdas

See [this page](http://en.cppreference.com/w/cpp/compiler_support) for extreme detail.

There is some discussion here: http://discourse.mc-stan.org/t/moving-to-c-11-again/75/21
See also https://cran.r-project.org/doc/manuals/r-release/R-exts.html#Using-C_002b_002b14-code for how to emulate features that the compiler is missing. And [here](https://github.com/stan-dev/math/pull/944) and [here](http://discourse.mc-stan.org/t/one-compiler-per-os/4899/) for more discussion on how we got to this testing arrangement.


## Automated style checking tools

### cpplint
`cpplint.py` is a Python (2.7) script that checks for these guidelines. Here is a make target that will check the source code for any instances that do not conform to this standard:

```
> make cpplint
```

If your default python version is python 3, the cpplint.py script fails silently. Once you install python 2.x, you can add this to your `make/local` file to use python 2.x to run the python script:
```
RUN_CPPLINT = python2 stan/lib/cpplint_4.45/cpplint.py
```

The make target includes Stan-specific options and limits the tests to the src/stan directory.

### clang-format
clang-format is a tool that will automatically format C++ code according to formatting rules in the `.clang-format` file. Our file is just a few changes from the vanilla Google Style guide. `clang-format` doesn't deal with many of our exceptions to the Style guide, below, as they are difficult to automate or not strictly formatting related.

#### Mac Setup
First, [install `clang-format`](http://geant.cern.ch/content/clang-format-git-hook). If you're on a mac and using homebrew, please remove any clang-formats you have installed and then install clang-format@2017-11-14 with this command:
```
brew install https://raw.githubusercontent.com/Homebrew/homebrew-core/5d29e8a6bfbe2e7d8f9525645bb0c00879d0984a/Formula/clang-format.rb
```
(if that doesn't work, try `brew install clang-format` or `brew upgrade clang-format` and get whatever your computer thinks is the latest)

#### Linux setup
If you're on Ubuntu Linux, you can use if you're on Ubuntu you can use the LLVM PPA to install clang-format-5.0 and then use update-alternatives to set it as the default. Find the relevant 5.0 PPA for your version of Ubuntu here: https://apt.llvm.org/ and follow their PPA install instructions, then:
```
sudo apt-get install clang-format-5.0
sudo update-alternatives --install /usr/bin/clang-format clang-format /usr/bin/clang-format-5.0 100
```

For other distributions, try installing from source: https://github.com/llvm-mirror/clang/commits/release_50

##### Editor setup
Emacs - there's a plugin/script called `google-c-style`. You can find it [here](https://raw.githubusercontent.com/google/styleguide/gh-pages/google-c-style.el) or just install from MELPA.

Spacemacs - you can add `google-c-style` to `dotspacemacs-additional-packages`.

Vim- check out https://github.com/google/vim-codefmt

##### Git hook
There is a pre-commit hook in hooks/pre-commit, and if you run `bash hooks/install_hooks.sh` it will be installed for you. Going forward, it will run `clang-format -i` (which implicitly uses the `.clang-format` file in the repo) on any changed files to format them. If you need to disable this for some reason (though your tests will fail if you haven't formatted things correctly) you can use `git commit --no-verify`.

##### Differences from clang-format's `--style=google`
To see which rules are different, you can run this command:
```sh
clang-format --style=google -dump-config > google
diff google .clang-format 
```
And here's the current output:
```
15,16c15,16
< AllowShortIfStatementsOnASingleLine: true
< AllowShortLoopsOnASingleLine: true
---
> AllowShortIfStatementsOnASingleLine: false
> AllowShortLoopsOnASingleLine: false
36c36
< BreakBeforeBinaryOperators: None
---
> BreakBeforeBinaryOperators: All
86c86
< SortIncludes:    true
---
> SortIncludes:    false
```

You can see the rule definitions [here.](https://clang.llvm.org/docs/ClangFormatStyleOptions.html) 

### iwyu

Include what you use testing.

#### Installation on Mac
Easy installation requires Homebrew.

Steps:

1. `brew tap jasonmp85/iwyu`
2. `brew install iwyu`

More details: https://github.com/jasonmp85/homebrew-iwyu


#### Running include-what-you-use on the parser
Once iwyu has been installed:
```
make -k test/test-models/stanc CC=iwyu &> iwyu.out
```

Inspect `iwyu.out`.

More info:
- `-k` flag continues running make in the presence of errors. iwyu always returns an error code, so omitting that will stop at the first compilation.
- `CC=iwyu` just replaces the compiler with iwyu. iwyu doesn't support the full set of compiler flags, so it'll put up warnings about some of our compiler flags.
- `&> iwyu.out` just grabs the output in both the stdout and stderr and puts it into iwyu.out.



## Code Quality, Documentation, and Testing

We want to institute some new policies on code, documentation, and testing quality. 

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

7.  Tests should be parallel in paths to the source files (`src/stan/**` and `test/unit/**`).

8. Use the nested namespace `internal` for hidden implementation details for developers.  The `internal` code has similar quality requirements but doesn't have the same completeness requirements for docs and testing that API functions have. 


## Unit testing
We know there are many bugs in our code.  With good test coverage, when a bug is found, these tests allow us to demonstrate:
 - That the bug exists,
 - that the bug has been fixed,
 - and, that the fix has won't break something else.

While we can't prove that the fix hasn't introduced new, unknown bugs, we can prove that this fix won't cause something else to break.  

To show that the bug exists, write a test that tickles the bug.  Before going on to fix the bug, run the test, verify that it causes failure, and add and commit the test to the Stan repo.

Now fix the bug.  To verify that the bug has been fixed, re-run the test and verify that it no longer fails.

Verify that the fix hasn't broken existing code.  To do this, we re-run existing tests.  Before submitting a pull request, we expect the developer to run those tests which directly exercise the code.  At a minimum, if you fix a bug in `stan/foo/bar.hpp`, then you should run all tests for `src/stan/foo`.

When a pull request is submitted, the entire Stan test suite will be run on our [Continuous Integration](https://github.com/stan-dev/stan/wiki/Continuous-Integration) server.

### How to invoke tests.

Stan tests code live in the directory `src/test` in the Stan repository.

Stan currently uses the Unix `make` utility, specifically GNU Make 3.8, to build and test Stan.  See [Testing Stan using GNU Make and Python](https://github.com/stan-dev/stan/wiki/Testing-Stan-using-Gnu-Make-and-Python) for details.  You can run `make help` to see some of the options.






## 3. Technical details. (Google test; our distribution testing framework)

We'll try to describe some of the implementation details of the tests for Stan.

- Test naming convention

    All tests are named `*_test.cpp` and are located in subdirectories of `src/test`.

    The test executable for `src/test/*_test.cpp` will be created at `test/*` (add `.exe` for Windows). The rule is to strip the leading `src/` and the `_test.cpp` from the filename.

- How to run a specific test

    The `make` target is test executable. For example, if the test is located at `src/test/foo/bar_test.cpp`, the command to build the test is ```> make test/foo/bar```.



## 4. If providing a new feature, what's the minimum level of testing required?
## 5. Legacy instructions.


- How to add new tests
- How the autogenerated tests work
- What tests to add for a new file?
- To what extent should methods be tested?
- Examples
