Welcome to Stan!  We're excited that you're interested in contributing to the project.  Before you're able to contribute, there are some processes and other information that are good to know.

The Stan project is [hosted on GitHub](https://github.com/stan-dev) so you will have to create a GitHub account if you do not yet already have one. Developer discussions are [hosted on Discourse](http://discourse.mc-stan.org) so you will have to create an account there in order to ask questions or participate in discussions.

Most of the following discussion is aimed at people who want to contribute code to Stan. But there are [many other ways to contribute!](https://github.com/stan-dev/stan/wiki/Contributing-to-Stan-Without-C-Plus-Plus--Experience)

## Project layout

You can see the overarching Stan project structure [here](https://github.com/stan-dev/stan/wiki#github-repositories-and-submodule-relationships). Each of the repos can be worked on independently, though some will include others as git submodules if they are dependent. Each of the repos also has their own wiki! Don't forget to check that wiki homepage and search it for information that might be related to that subproject.

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
* [Agner Fog's manuscript](http://www.agner.org/optimize/optimizing_cpp.pdf)