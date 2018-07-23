# Tested Toolchains and OSes
### On PRs:
* mac / clang 6 / libc++ Threading
* linux / gcc 5 / stdlibc++ OpenMPI
* mac clang 6 libc++ OR linux clang 6 libc++ distribution tests (without row vectors)

### Additional tests on merge to develop

* linux / gcc 5 / stdlibc++ Threading
* mac / clang 6 libc++ OR linux / gcc 5 stdlibc++ GPU
* mac clang 6 libc++ OR linux clang 6 stdlibc++ distribution tests (with row vectors)

So there will be a little bit of non-determinism for the sake of running tests more quickly on available hardware.


# C++ Standard library version
If you're on Mac, this is probably libc++, and on Windows or Linux this is probably GNU's stdlibc++. We should support both for the versions of compilers we support.

# C++ Language Features
### [C++11 features](http://blog.smartbear.com/c-plus-plus/the-biggest-changes-in-c11-and-why-you-should-care/)
* everything except for Expression SFINAE
### C++14 features
* auto and decltype(auto) return types
* generic lambdas

See [this page](http://en.cppreference.com/w/cpp/compiler_support) for extreme detail.

There is some discussion here: http://discourse.mc-stan.org/t/moving-to-c-11-again/75/21
See also https://cran.r-project.org/doc/manuals/r-release/R-exts.html#Using-C_002b_002b14-code for how to emulate features that the compiler is missing. And [here](https://github.com/stan-dev/math/pull/944) and [here](http://discourse.mc-stan.org/t/one-compiler-per-os/4899/) for more discussion on how we got to this testing arrangement.