# Guidelines
Stan is primarily a C++ project, but the ideas below vary in the degree to which skill with C++ is required from "none at all" to "familiarity with Boost or Eigen". Stan is hosted on GitHub, so some experience with the git version-control system is a plus but git can be learned in a relatively short period of time.

Stan consists of a few different parts. The modeling language allows users to specify a Bayesian model for data. The parser inputs a file in the modeling language and outputs a C++ source file, which can then be compiled into an executable file that links to the library. Data are passed to this executable, which outputs results to the hard disk. These results can then be summarized.  

# Ideas
## Interfaces to Stan
There are three interfaces to Stan currently. The first is the command-line of a shell. The second is via the R package RStan. The third is via the Python module PyStan. Additional interfaces are welcome. The easiest approach is to use the calling program to write the data to the hard disk, use a shell call to the command-line interface to Stan, and read the results into the calling program. The harder approach (which is undertaken by RStan and PyStan) is use the calling program to pass the data-in-memory to Stan and accept the results-in-memory from Stan. For a GSOC project, we favor starting with the first approach and implementing the second approach if there is time.

### HTML5 Interface using PNaCL (mentor: Ben Goodrich)
The Portable Native Client (PNaCL) compiler takes C++ source code and compiles it into an intermediate representation that can be executed in a sandbox on a variety of platforms via the Chrome web browser. This mechanism makes it possible for users to distribute the intermediate representation of their Bayesian models for others to execute without additional compilation. We have already used the PNaCL compiler to create the intermediate representation but would greatly benefit from a GSOC project to write an HTML5 interface that would pass a data file and various options to the executable and pass results to the executable that prints a summary. This project would require no C++ but would require intermediate web design skill, including javascript and HTML5. See the [Native Client](https://developers.google.com/native-client/dev/) website for more details on PNaCL in particular the last few steps of the [tutorial](https://developers.google.com/native-client/dev/devguide/tutorial/tutorial-part1).

### HTML5 Interface using emscripten (mentor: Ben Goodrich)
The emscripten compiler takes C++ source code and translates it into javascript that can be run on a variety of platforms via many web browsers. The development version of emscripten uses a fork of the PNaCL compiler to create the intermediate representation and generates the resulting javascript. We would benefit from a GSOC project to write an HTML5 interface that would pass a data file and various options to the executable and pass results to the executable that prints a summary. This project would require no C++ but would require intermediate web design skill, including javascript and HTML5. See the [emscripten](https://github.com/kripken/emscripten/wiki) website for more details on this process.

### Java Interface (mentor: TBD)
Bob said in an email:
> Having just said this, it occurs to me that having a Java
> and Scala wrapper for Stan along the lines of PyStan and
> RStan (and soon to be MStan for MATLAB) would make sense as
> a project.

> The performance hit from translating data structures shouldn't
> be an issue for sampling or optimization.

> With a wrapper, it'd be possible to experiment with other algorithms
> by calling the log probability function compiled in C++ for
> a model.  I've never used JNI, but it may be a bottleneck for
> a tighter integration like this as the parameters would need to
> be passed around for each call and the result passed back:

>   [http://stackoverflow.com/questions/7699020/what-makes-jni-calls-slow](http://stackoverflow.com/questions/7699020/what-makes-jni-calls-slow)

### Julia Interface (mentor: TBD)
### Stata Interface (mentor: TBD)
### Matlab Interface (mentor: TBD)
A simple version is already in-progress. It may be worthwhile for a GSOC project to implement a more complicated in-memory interface. Doing so would require at least intermediate skill with Matlab and C++ programming.

## Input / Output
The command-line interface to Stan currently only reads data from the disk in a format that is heavily influenced by BUGS and R. Also, the command-line interface to Stan only writes results to the disk in CSV format, although CSV files can be read by a wide variety of statistical programs. There have been many requests for additional input / output options, any of which would make a good GSOC project. In each case, some skill with C++ is necessary but the main requirement is experience with the input format.

### JSON Input (mentor: TBD)
[Stan-users discussion](https://groups.google.com/forum/?fromgroups=#!searchin/stan-users/JSON/stan-users/gtK7ULwodc4/j-yJoDna7O4J) 

### CSV Input
[Issue 421](https://github.com/stan-dev/stan/issues/421)

### File-backed Input
[Stan-dev discussion](https://groups.google.com/d/msg/stan-dev/WM0FnSHeUx8/tYkMuy_JU-gJ)

### Protobuf Input
[Link](http://code.google.com/p/protobuf/)

## Statistical Computing
These projects enable users to estimate new models with Stan or to estimate existing models in more computationally efficient ways. As such, they all require at least intermediate C++ skill and some knowledge (or capacity to learn) the theory behind the algorithm.

### Initialization block (mentor: TBD)
A Stan model requires starting values for all the unknown parameters. These starting values can be input from a file on the hard disk, but many users find doing so inconvenient. A better alternative would be to add an initialization block to the Stan modeling language that could be parsed into C++ code, which would allow users to specify particular starting values for (a subset of) the parameters or to specify a statistical distribution to randomly draw the corresponding starting values. Stan already has the capability to draw randomly from many statistical distributions but lacks the capability to utilize them for starting values.

### Limited memory BFGS optimization (mentor: TBD)
Stan already has two optimization algorithms, an Newton-based approach and a quasi-Newton approach. Both require that a (approximation to the) Hessian matrix of the unknown parameters be allocated, which limits the applicability to optimization problems with many parameters. An alternative is to use some "limited memory" variant of the BFGS algorithm. These limited memory alternatives are already available in a variety of software, but Stan is somewhat unique in that it does not require the user to implement a function to calculate the gradient since the gradient can be calculated via auto-differentiation. Thus, having a limited memory optimization algorithm would be quite useful for Stan. See the [mentions](http://books.google.com/books?id=VbHYoSyelFcC&lpg=PP1&dq=Nocedal%20Wright&pg=PP1#v=onepage&q=%22L-BFGS%22&f=false) of "L-BFGS" in Nocedal and Wright's book _Numerical Optimization_.

### Conditionally independent contributions to the log-posterior (mentor: TBD)
The Stan modeling language currently supports statements such as `vector ~ distribution(parameters);` which (when appropriate) indicates the the elements of the `vector` are conditionally independent given the `parameters` (which are often scalars) to the `distribution` and thus contribute additively to the log-posterior that is essential to the Markov Chain Monte Carlo (MCMC) schemes used by Stan. However, many users find this syntax is still too limiting and Stan would benefit from a GSOC project to support `array ~ distribution(parameters);` where `array` is a multidimensional object such as a matrix-like two-dimensional object. This syntax would indicate the user's intention that all elements of the `array` are conditionally independent given the `parameters` to the `distribution`. Permitting this syntax is relatively easy, but the harder part is implementing the desired behavior for as many `distributions` as possible that are already supported by Stan.

### Improvement of multivariate distributions (mentor: Ben Goodrich)

### Additional statistical distributions (mentor: TBD)
Stan already makes it possible for users to use dozens of statistical distributions in their models, but perhaps the most common request is for additional ones. We would be especially interested in a GSOC project that implemented more general statistical distributions that include many well-known distributions as special cases. Or if the student would be more comfortable implementing a few well-known statistical distributions, that would be great as well.