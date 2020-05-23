We generally follow the [Google Style Guide](https://google.github.io/styleguide/cppguide.html) except for the exceptions listed in the following sections. We use a few tools to automate style and formatting checking, but they don't completely capture our formatting and style guidelines.

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

## Code Quality

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