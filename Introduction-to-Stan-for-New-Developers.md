Welcome to Stan!  We're excited that you are interested in contributing to the project.  Before you can contribute you will have to familiarize yourself with the particular processes for new contributions that we have incorporated to facilitate the growth of the project.

Firstly, the Stan project is [hosted on GitHub](https://github.com/stan-dev) so you will have to create a GitHub account if you do not yet already have one.  Secondly, developer discussions are [hosted on Discourse](http://discourse.mc-stan.org) so you will have to create an account there in order to ask questions or participate in discussions.

Every developer has their own local development setup, but we have compiled [various helpful ](https://github.com/stan-dev/stan/wiki/Dev:-Tricks) that you might find useful.

## Style

In order to ensure that we can quickly read and understand contributions, consistent style is incredibly important.  We have adopted conventions for [code quality](https://github.com/stan-dev/stan/wiki/Code-Quality) and [code style](https://github.com/stan-dev/stan/wiki/Coding-Style-and-Idioms) to which all contributions must conform.

## Testing

The robustness of Stan is only as good as our test coverage, and we require that all new contributions are adequately tested.  We use the [GoogleTest framework](https://github.com/stan-dev/stan/wiki/How-to-Write-Unit-Tests-with-GoogleTest) for writing tests and [GnuMake and Python](https://github.com/stan-dev/stan/wiki/Testing-Stan-using-Gnu-Make-and-Python) for running those tests.

## Contributing to Stan

We have adopted the GitFlow process for incorporating new contributions into Stan.  If you are not yet familiar with Git we recommend that you check out many of the great Git tutorials freely available online.  Once you are comfortable with Git itself you can read about are particular implementation of GitFlow [here](https://github.com/stan-dev/stan/wiki/Dev:-Git-Process) and [here](https://github.com/stan-dev/stan/wiki/Developer-process-overview).  

All new contributions are also tested with out [continuous integration framework](https://github.com/stan-dev/stan/wiki/Testing:-Continuous-Integration).

## Contributing Core Code

The core code in Stan is written in heavily-templated C++ to ensure high-performance.  There are many great C++ tutorials available online, for example [cplusplus.org](http://www.cplusplus.com/doc/tutorial/), and once you are familiar with the basics of the language you can tackle the subtleties of templates.  We highly recommend [Vandevoorde and Josuttis](https://www.amazon.com/Templates-Complete-Guide-David-Vandevoorde/dp/0201734842) and [Alexandrescu](https://www.amazon.com/Modern-Design-Generic-Programming-Patterns/dp/0201704315).

There are many additional resources available for learning how to optimize C++ code, including [Agner Fog's manuscript](http://www.agner.org/optimize/optimizing_cpp.pdf) and the many books of, amongst others, Scott Meyers and Herb Sutter.

## Contributing to the Interfaces

The Stan interfaces wrap the core C++ code and expose its functionality to other languages, such as R and Python.  Consequently contributions to the interfaces may require knowledge of how to couple these languages together, for example with Rccp and Cython, or be built entirely in the interface language.  For details on a specific interface please consult the corresponding GitHub repository.

Once you have familiarized yourself with our process take a look at the GitHub issue trackers for the many tasks that need to be tackled!  We look forward to hearing from you on Discourse and seeing your pull requests!