Welcome to Stan!  We're excited that you're interested in contributing to the project.  Before you're able to contribute, there are some processes and other information that are good to know.

The Stan project is [hosted on GitHub](https://github.com/stan-dev) so you will have to create a GitHub account if you do not yet already have one. Developer discussions are [hosted on Discourse](http://discourse.mc-stan.org) so you will have to create an account there in order to ask questions or participate in discussions.

Most of the following discussion is aimed at people who want to contribute C++ code to Stan. But there are [many other ways to contribute that don't involve C++](https://github.com/stan-dev/stan/wiki/Contributing-to-Stan-Without-C-Plus-Plus--Experience)!


## Licensing

We're committed to having a permissive open-source license. The stan-dev/math, stan-dev/stan, and stan-dev/cmdstan libraries are [licensed with the BSD 3-Clause License](https://github.com/stan-dev/math/blob/develop/LICENSE.md) and we only accept changes to the code base that compatible with this license.

When you write code, you own the copyright to your code unless you've assigned it to another entity, such as your employer.  Contributions to Stan leave copyright ownership with the contributor or their assignee.  Per the [GitHub terms of service](https://docs.github.com/en/github/site-policy/github-terms-of-service) (see the section, User-Generated Content, any code contributed to Stan through GitHub is released under the same license as the repository to which it is contributed. 


## Issues

We reserve [issues](https://github.com/stan-dev/math/issues) for bugs and feature requests that are defined well enough for a developer to tackle. If you have general questions about the Math library, please see the [Discussion](#discussion) section.

### Bug Reports

Ideally, bug reports will include these pieces of information:

1. a description of the problem
2. a reproducible example
3. the expected outcome if the bug were fixed.

If there's an error and you can produce any of these pieces, it's much appreciated!


### Feature Requests

We track the development of new features using the same [issue tracker](https://github.com/stan-dev/math/issues). Ideally, feature requests will have:

1. a description of the feature
2. an example
3. the expected outcome if the feature existed.

Open feature requests should be the ones we want to implement in the Math library. We'll try to close vague feature requests that don't have enough information and move the discussion to the forums.

## Pull Requests

All changes to the Math library are handled through [pull requests](https://github.com/stan-dev/math/pulls). Each pull request should correspond to an issue. We follow a [modified GitFlow branching model](https://github.com/stan-dev/stan/wiki/Dev:-Git-Process) for development.

When a contributor creates a pull request for inclusion to the Math library, here are some of the things we expect:

1. the contribution maintains the Math library's open-source [license](https://github.com/stan-dev/math/wiki/Developer-Doc#licensing): 3-clause BSD
2. the code base remains stable after merging the pull request; we expect the `develop` branch to always be in a good state
3. the changes are maintainable. In code review, we look at the design of the proposed code. We also expect documentation.
4. the changes are tested. For bugs, we expect at least one test that fails before the patch and is fixed after the patch. For new features, we expect at least one test that shows expected behavior and one test that shows the behavior when there's an error.
5. the changes adhere to the Math library's [C++ standards](https://github.com/stan-dev/stan/wiki/Code-Quality). Consistency really helps.

Pull requests are code reviewed after they pass our continuous integration tests. We expect all the above before a pull request is merged. We are an open-source project and once code makes it into the repository, it's on the community to maintain.

### Code Reviews
See the [Code Review Guidelines](https://github.com/stan-dev/math/wiki/Developer-Doc#code-review-guidelines) on the Math wiki.


## Discussion

For general questions, please ask on the forums with the ["Developers" tag](http://discourse.mc-stan.org/c/stan-dev).


## GitHub repositories and submodule relationships

The development for the math library, language and algorithms, and interfaces are arranged into the following repositories with arrows indicating submodule inclusions.

```      
math <- stan <- pystan
             <- rstan   <- rstanarm
             <- cmdstan <- statastan
                        <- matlabstan
                        <- stan.jl
                        <- MathematicaStan
     <- stanc3
```

Currently, the `stan` repo includes the algorithms and the service API for the interfaces.  There are additional repos for tools such as the emacs mode, R plotting, R Shiny interface, web pages, etc. `stanc3` hosts the language compiler.   Some of the modules include others as submodules if there's a code dependency.  Each of the repos also has their own wiki! Don't forget to check that wiki homepage and search it for information that might be related to that subproject.

## Contributing to Stan

### The Lifecycle of a Pull Request
People use Stan in many contexts. When we're deciding how to add/remove/modify Stan code, we need to understand what our goals are. This typically involves some discussion where we try to elicit some concrete use-cases for the feature, followed by a github issue in the appropriate repo with something resembling a spec for the issue that the reviewer can use to evaluate an associated pull request, followed by that pull request. These three artifacts exist in different locations, so at the top of each one there should be a link to the others and an attempt to summarize the results of previous steps in the workflow. To summarize:

1. Bring up your proposed feature for discussion on [our forums](http://discourse.mc-stan.org). If you're trying to find a place to help out, you can skip this and find an existing issue on the appropriate github repo.
1. Summarize the discussion and write something approaching a high level spec in a github issue.
1. Create a pull request with an attempt to address a github issue. 

You can read more about the developer process [here](https://github.com/stan-dev/stan/wiki/Developer-process-overview).

### Misc
We have adopted the GitFlow process for incorporating new contributions into Stan.  If you are not yet familiar with Git we recommend that you check out many of the great Git tutorials freely available online.  Once you are comfortable with Git itself you can read about are particular implementation of GitFlow [here](https://github.com/stan-dev/stan/wiki/Dev:-Git-Process) and [here](https://github.com/stan-dev/stan/wiki/Developer-process-overview).  

All new contributions are also tested with out [continuous integration framework](https://github.com/stan-dev/stan/wiki/Testing:-Continuous-Integration).

Every developer has their own local development setup, but we have compiled [various helpful tricks](https://github.com/stan-dev/stan/wiki/Dev:-Tricks) that you might find useful.

### Style
In order to ensure that we can quickly read and understand contributions, consistent style is incredibly important.  We have adopted conventions for [code quality](https://github.com/stan-dev/stan/wiki/Code-Quality) and [code style](https://github.com/stan-dev/stan/wiki/Coding-Style-and-Idioms) to which all contributions must conform. You can read more on these links, but we use an automated formatter for many of our conventions.

There is a list of supported compilers and language features [here](https://github.com/stan-dev/stan/wiki/Supported-C---Compilers-and-Language-Features).

### Testing

The robustness of Stan is only as good as our test coverage, and we require that all new contributions are adequately tested.  We use the [GoogleTest framework](https://github.com/stan-dev/stan/wiki/How-to-Write-Unit-Tests-with-GoogleTest) for writing tests and [GnuMake and Python](https://github.com/stan-dev/stan/wiki/Testing-Stan-using-Gnu-Make-and-Python) for running those tests.

### Documentation
We have two main sources of documentation - Doxygen doc comments and the Stan manual. You can read more about contributing to the former [here](https://github.com/stan-dev/stan/wiki/How-to-Write-Doxygen-Doc-Comments). The latter typically has a github issue for each Stan release associated with it on the Stan repo, but we also take pull requests to the .tex files. 

There are other forms of documentation listed on the website [here](http://mc-stan.org/users/documentation/).

### Contributing Core Code

Much of what you might consider to be the "core" of Stan actually exists in [the Math repo](https://github.com/stan-dev/math). This document applies to that repo, but you can read more about how that repo is organized and any differences [here](https://github.com/stan-dev/math/wiki/Introduction-to-Stan-Math-for-New-Developers/).

The core code in Stan is written in heavily-templated C++ to ensure high-performance.  There are many great C++ tutorials available online, for example [cplusplus.org](http://www.cplusplus.com/doc/tutorial/), and once you are familiar with the basics of the language you can tackle the subtleties of templates.  We highly recommend [Vandevoorde and Josuttis](https://www.amazon.com/Templates-Complete-Guide-David-Vandevoorde/dp/0201734842) and [Alexandrescu](https://www.amazon.com/Modern-Design-Generic-Programming-Patterns/dp/0201704315).

There are many additional resources available for learning how to optimize C++ code, including [Agner Fog's manuscript](http://www.agner.org/optimize/optimizing_cpp.pdf) and the many books of, amongst others, Scott Meyers and Herb Sutter.

### Contributing new densities

Having a comprehensive set of useful densities coded in the Stan math library is a benefit to users.  Densities are also a maintenance burden both for testing and for understanding the code base.  As a result we are somewhat cautious about including new densities. Guidelines for including densities:

- The pdf, cdf, and rng should be available so users of the Stan language don't need to check the manual.
- There should be a computational benefit to coding the density in C++.  Some densities can easily and efficiently be specified in the Stan language and the benefits of coding them in C++ are limited.  It helps to provide some evidence of the computational benefits.
- The density should be applicable to a range of problems.
- If the density's C++ code re-implements or improves on functions already present in the math library, the necessary improvements should be coded separately in the math library.
- Ongoing interest from the code author in maintaining the code.


### Contributing to the Interfaces

The Stan interfaces wrap the core C++ code and expose its functionality to other languages, such as R and Python.  Consequently contributions to the interfaces may require knowledge of how to couple these languages together, for example with Rccp and Cython, or be built entirely in the interface language.  For details on a specific interface please consult the corresponding GitHub repository.

Once you have familiarized yourself with our process take a look at the GitHub issue trackers for the many tasks that need to be tackled!  We look forward to hearing from you on Discourse and seeing your pull requests!

## Useful utilities

This section contains some tips for using developer-oriented tools and setting up a computing environment for development.

### Git

#### Remove Untracked Files

There are a lot of files that are in our `.gitignore` file that stack up and don't gt cleaned.  In order to remove every untracked file, including hidden ones, do this:

```
git clean -d -x -f
```

**Warning:** this will kill everything that's not currently being tracked.  You probably want to run `git status` first.

### Git completion
https://github.com/git/git/tree/master/contrib/completion

* git-prompt.sh. This changes the prompt on the command line to show the current branch. Install by copying it to `~/`, follow install instructions. For a cleaner prompt, replace the PS1 suggested with: 
`PS1='\w$(__git_ps1 " (%s)")> '`
The prompt will look like:
`~/stan (master)> `
where "~/stan" is the current directory, "(master)" indicates the current branch is the master branch.

* git-completion.*sh. Install this for auto-completion from the command line. It auto-completes git commands and git branches. For example, type `git checkout ` then hit tab twice. It should show the available branches.

### Mac utilities

#### Aquamacs: single kill buffer

By default, aquamacs will has multiple kill buffers. This means that there is a copy and paste buffer by using command-c/x/v and there is a separate copy and paste buffer by using ctrl-w, alt-w, ctrl-y. This gets really confusing. Here's how to have a single kill buffer so copying from any Mac program will paste into emacs using ctrl-y or command-v.

1. Open ~/.emacs (or wherever else you're storing your preferences)
2. Add: `(setq x-select-enable-clipboard t)`

#### Fixing Line Feeds and Tabs

Ant includes a handy task called  [FixCRLF](http://ant.apache.org/manual/Tasks/fixcrlf.html) that "Adjusts a text file to local conventions."  So you can set it to replace tabs with spaces and Windows line ends with unix ones.

### Emacs

#### Autoformatting space

To make sure you never have tabs or line-ending spaces in your code files, you can use this in your ```.emacs``` file:

```
(defun java-mode-untabify ()
   (save-excursion
     (goto-char (point-min))
     (if (search-forward "\t" nil t)
         (untabify (1- (point)) (point-max))))
   nil)

 (add-hook 'java-mode-hook
           '(lambda ()
              (make-local-variable 'write-contents-hooks)
              (add-hook 'write-contents-hooks 'java-mode-untabify)))

 (add-hook 'html-mode-hook
           '(lambda ()
              (make-local-variable 'write-contents-hooks)
              (add-hook 'write-contents-hooks 'java-mode-untabify)))

 (add-hook 'cpp-mode-hook
           '(lambda ()
              (make-local-variable 'write-contents-hooks)
              (add-hook 'write-contents-hooks 'java-mode-untabify)))

 (add-hook 'stan-mode-hook
           '(lambda ()
              (make-local-variable 'write-contents-hooks)
              (add-hook 'write-contents-hooks 'java-mode-untabify)))
```

You can rename --- the "Java" in the title is a holdover from where I first got the macros.

You can also automatically remove line-final whitespace (this is just for C++, but it could be hooked elsewhere):

```
(add-hook 'c++-mode-hook
          (lambda () (add-to-list 'write-file-functions 'delete-trailing-whitespace)))

```

### Cpp

#### OS Detection

Not just for Windows:

http://nadeausoftware.com/articles/2012/01/c_c_tip_how_use_compiler_predefined_macros_detect_operating_system
Some GCC specifics:

http://stackoverflow.com/questions/259248/how-do-i-test-the-current-version-of-gcc

## Stan developer resources
* [Stan project structure](https://github.com/stan-dev/stan/wiki#github-repositories-and-submodule-relationships)
* [Stan Users Group](https://groups.google.com/forum/#!categories/stan-users/general)
* [Discourse](http://discourse.mc-stan.org/)
* [Developer process](https://github.com/stan-dev/stan/wiki/Developer-process-overview)
* [Contributing a new Stan function](https://github.com/stan-dev/stan/wiki/Contributing-New-Functions-to-Stan)
* [Stan C++ style guide](https://github.com/stan-dev/stan/wiki/Coding-Style-and-Idioms) (includes some developer environment setup)
* [Autodiff paper](https://arxiv.org/abs/1509.07164) Details the implementation and math library generally
* [Some Bayesian Modeling Techniques in Stan](https://www.youtube.com/watch?v=uSjsJg8fcwY)

## C++ resources
* [Vandevoorde's C++ Templates] (http://www.josuttis.com/tmplbook/)
 and the free PDF here: [Vandevoorde's Templates PDF](http://ultra.sdk.free.fr/docs/DxO/C%2B%2B%20Templates%20The%20Complete%20Guide.pdf)
* [Alexandrescu's Modern C++ Design] (http://erdani.com/index.php/books/modern-c-design/)
and the free PDF here: [Modern C++ PDF](http://index-of.co.uk/C++/C++%20Design%20Generic%20Programming%20and%20Design%20Patterns%20Applied.pdf)
* [Agner Fog's C++ optimization tips](http://www.agner.org/optimize/optimizing_cpp.pdf)
* [On template metaprograms](https://www.fluentcpp.com/2017/06/02/write-template-metaprogramming-expressively/)

# Math library overview

The Stan Math library, referred to as Math or the Math library, is a C++ library for automatic differentiation. It's designed to be usable, extensive and extensible, efficient, scalable, stable, portable, and redistributable in order to facilitate the construction and utilization of algorithms that utilize derivatives.

The Math library implements:

- [reverse mode](https://en.wikipedia.org/wiki/Automatic_differentiation#Reverse_accumulation) automatic differentiation for computing gradients. This is fully tested and is utilized by [Stan](https://github.com/stan-dev/stan).
- [forward mode](https://en.wikipedia.org/wiki/Automatic_differentiation#Forward_accumulation) automatic differentiation for computing directional derivatives.  This does not work for higher-order functions, but is otherwise completely tested.
- mixed mode automatic differentiation for computing higher order derivatives. Once forward mode is fully tested, this should work.

Some key features of the Math library's reverse mode automatic differentiation:
- object oriented design with overloading of operators
- arena-based memory management

For implementation details of the Math library's automatic differentiation, please read the arXiv paper "[The Stan Math Library: Reverse-Mode Automatic Differentiation in C++](https://arxiv.org/abs/1509.07164)."

# FAQ

## Why do we have `BOOST_DISABLE_ASSERTS` included in the C++ build?

Without including this, Boost will assert if certain inputs do not meet the preconditions of the function. Assertions are difficult to trap and recover from and we want to continue to have control over this behavior.

See [Discourse: Boost defines](https://discourse.mc-stan.org/t/boost-defines/10087) for more details.

