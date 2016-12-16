# DRAFT 12/16/2016

# Project layout
[Stan Project Structure](https://github.com/stan-dev/stan/wiki#github-repositories-and-submodule-relationships)

Each of the repos should be able to be worked on independently. The ones with dependencies include them as git submodules.

# How to contribute
Try picking an issue off the github issues for the relevant repo. Math is potentially a good place to start as it has no dependencies. Generally speaking the "develop" branch should be stable and passing the build and tests. You should begin your changes there. For more details see the developer process in the links.

# Getting help
Try emailing the [Stan Users Group](https://groups.google.com/forum/#!categories/stan-users/general) or signing up and posting on [Discourse](http://discourse.mc-stan.org/)
Currently you have to email someone on the dev team to grant you access to post in those places, but don't be shy! 

# Concepts to know
* Git (basics)
* Github (also basics; cloning, using the issue tracker)
* C++, especially template metaprogramming

# More resources
* [Github tutorial](https://guides.github.com/activities/hello-world/) 
* [Developer process](https://github.com/stan-dev/stan/wiki/Developer-process-overview)
* [Contributing a new Stan function](https://github.com/stan-dev/stan/wiki/Contributing-New-Functions-to-Stan)

# Recommended reading
* [Autodiff paper](https://arxiv.org/abs/1509.07164) Details the implementation and math library generally
* [Vandevoorde's C++ Templates] (http://www.josuttis.com/tmplbook/)
* [Alexandrescu's Modern C++ Design] (http://erdani.com/index.php/books/modern-c-design/)
