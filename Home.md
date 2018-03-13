## Welcome!

This is the top-level Wiki for the Stan project as well as the Wiki for the stan-dev/stan repository.  Most of the content is aimed at developers, though some may be of interest to users.  If you're not a Stan developer or thinking of becoming one, you're probably looking for:

* [Stan Home Page](http://mc-stan.org/)

There are a few remaining user-facing Wiki pages that haven't been moved to the web site:

* [Prior choice](https://github.com/stan-dev/stan/wiki/Prior-Choice-Recommendations)
* [Where do I report bugs or request features?](https://github.com/stan-dev/stan/wiki/Where-do-I-create-a-new-issue)
* [Frequently encountered problems](https://github.com/stan-dev/stan/wiki/Frequently-Encountered-Problems)
* [Installing older versions](https://github.com/stan-dev/stan/wiki/Installing-Older-Versions-of-Stan-and-RStan)

## Introduction for aspiring Stan developers
See https://github.com/stan-dev/stan/wiki/Introduction-to-Stan-for-New-Developers

## To-Do List

* [Longer-term to-do items](https://github.com/stan-dev/stan/wiki/Longer-Term-To-Do-List)

## Stan Meetings

* [Dev meeting agenda](https://github.com/stan-dev/stan/wiki/Stan-Development-Meeting-Agenda)

## GitHub Repositories and Submodule Relationships

The development for the math library, language and algorithms, and interfaces are arranged into the following repositories with arrows indicating submodule inclusions.

```               
math <- stan <- pystan
             <- rstan   <- rstanarm
             <- cmdstan <- statastan
                        <- matlabstan
                        <- stan.jl
                        <- MathematicaStan
```

Currently, the `stan` repo includes the language, the algorithms, and the service API for the interfaces.  There are additional repos for tools such as the emacs mode, R plotting, R Shiny interface, web pages, etc.

## Developer Process

#### Process

* [Dev process overview](https://github.com/stan-dev/stan/wiki/Developer-process-overview)
* [Git process](https://github.com/stan-dev/stan/wiki/Dev:-Git-Process)

#### New Functions

* [Contributing new functions](https://github.com/stan-dev/stan/wiki/Contributing-New-Functions-to-Stan)

#### Code Style

* [Code quality](https://github.com/stan-dev/stan/wiki/Code-Quality)
* [Coding style and idioms](https://github.com/stan-dev/stan/wiki/Coding-Style-and-Idioms)
* [Doxygen for API doc](https://github.com/stan-dev/stan/wiki/How-to-Write-Doxygen-Doc-Comments)

#### Testing

* [Testing framework](https://github.com/stan-dev/stan/wiki/Testing-framework)
* [Testing tools and procedures](https://github.com/stan-dev/stan/wiki/Development-Testing:--Tools-and-Procedures)
* [Googletest for unit tests](https://github.com/stan-dev/stan/wiki/How-to-Write-Unit-Tests-with-GoogleTest)
* [Python unit test script](https://github.com/stan-dev/stan/wiki/Testing-Stan-using-Gnu-Make-and-Python)
* [Jenkins for continuous integration](https://github.com/stan-dev/stan/wiki/Jenkins)
* [Continuous integration (1)](https://github.com/stan-dev/stan/wiki/Continuous-Integration-Testing)
* [Continuous integration (2)](https://github.com/stan-dev/stan/wiki/Testing:-Continuous-Integration)

#### Miscellaneous

* [How to upgrade version numbers](https://github.com/stan-dev/stan/wiki/Process:-upgrade-version-numbers)


## Feature Specifications

#### Language

* [Ragged arrays](https://github.com/stan-dev/stan/wiki/Ragged-array-spec)
* [Tuple data type](https://github.com/stan-dev/stan/wiki/Functional-Spec:-List-Tuple-types)
* [Sparse matrix data type](https://github.com/stan-dev/stan/wiki/Functional-Spec:-Sparse-Matrix-Data-Types)
* [Functionals and higher-order functions](https://github.com/stan-dev/stan/wiki/Functionals-spec)
* [Stan 3 density notation and increments](https://github.com/stan-dev/stan/wiki/Stan-3-Density-Notation-and-Increments)
* [Parameter labeling](https://github.com/stan-dev/stan/wiki/Parameter-Labeling-Specification)
* [Parser pedantic mode](https://github.com/stan-dev/stan/wiki/Stan-Parser-Pedantic-Mode)

#### Interfaces

* [Services refactor](https://github.com/stan-dev/stan/wiki/Services-Refactor-Design-Document)
* [CloudStan](https://github.com/stan-dev/stan/wiki/CloudStan-functional-specification)
* [Logging](https://github.com/stan-dev/stan/wiki/Logging-Spec)
* [Log levels and consolidated output across interfaces](https://github.com/stan-dev/stan/wiki/Design:-Consolidated-Output-for-Sample,-Optimize,-ADVI,-and-Diagnose)
* [Stan 3 unified interface](https://github.com/stan-dev/stan/wiki/Stan-3-Unified-Interface)
* [Stan C++ API refactor](https://github.com/stan-dev/stan/wiki/Stan-Cpp-API-Refactor)

#### I/O

* [JSON I/O](https://github.com/stan-dev/stan/wiki/JSON-for-model-data-and-parameters)
* [Protocol buffer I/O](https://github.com/stan-dev/stan/wiki/Protocol-Buffers-for-serialization-of-input-data,-output-samples,-initial-values,-input-parameters,-and-output-messages,)
* [Output format](https://github.com/stan-dev/stan/wiki/Output-format)

#### Algorithms

* [Max (marginal) likelihood](https://github.com/stan-dev/stan/wiki/MLE-and-MML-Design)
* [ODE models](https://github.com/stan-dev/stan/wiki/Complex-ODE-Based-Models)


#### Documentation

* [Documentation reorganization](https://github.com/stan-dev/stan/wiki/Documentation-Organization)



## Developer Miscellany

* [Compiler and OS detection in C++](https://github.com/stan-dev/stan/wiki/Compiler-and-OS-Detection-Macros-for-Cpp)
* [C++ 11](https://github.com/stan-dev/stan/wiki/Cpp11-Upgrade)
* [Helpful tricks](https://github.com/stan-dev/stan/wiki/Dev:-Tricks)
* [Include what you use](https://github.com/stan-dev/stan/wiki/include-what-you-use)
* [Model C++ concept](https://github.com/stan-dev/stan/wiki/Model-Concept)
* [ODE integrator support](https://github.com/stan-dev/stan/wiki/ODE-Integrator-Support)
* [Developer UI guidelines](https://github.com/stan-dev/stan/wiki/User-Interface-Guidelines-for-Developers)
* [Yeti cluster at Columbia](https://github.com/stan-dev/stan/wiki/Yeti-Cluster)

## Advertising and Fundraising

* [Sell sheets](https://github.com/stan-dev/stan/wiki/Stan-Advertising-Webpages)

## Teaching

* [Some areas of pedagogy](https://github.com/stan-dev/stan/wiki/Pedagogy)