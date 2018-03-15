# Compilers
* [GCC 4.9.3+](https://gcc.gnu.org/gcc-4.9/changes.html) (4.9.3 is the version that ships with RTools 3.4, required for rstan windows support).
* [Microsoft Visual Studio 2015](https://msdn.microsoft.com/en-us/library/hh567368.aspx) (as required by PyStan on Windows)
* clang 3.4+

# C++ Standard library
If you're using clang, we'll try to use the llvm project's libc++, so that means you need to have libc++-dev (or equivalent package) installed. If you're using g++, we'll default to the gcc project's stdlibc++, which is usually included with g++.

# Language Features
### [C++11 features](http://blog.smartbear.com/c-plus-plus/the-biggest-changes-in-c11-and-why-you-should-care/)
* everything except for Expression SFINAE
### C++14 features
* auto and decltype(auto) return types
* generic lambdas

See [this page](http://en.cppreference.com/w/cpp/compiler_support) for extreme detail.

There is some discussion here: http://discourse.mc-stan.org/t/moving-to-c-11-again/75/21
See also https://cran.r-project.org/doc/manuals/r-release/R-exts.html#Using-C_002b_002b14-code for how to emulate features that the compiler is missing.