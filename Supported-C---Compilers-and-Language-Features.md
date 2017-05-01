# Compilers
* [GCC 4.9.3+](https://gcc.gnu.org/gcc-4.9/changes.html) (4.9.3 is the version that ships with RTools 3.4, required for rstan windows support).
* [Microsoft Visual Studio 2015](https://msdn.microsoft.com/en-us/library/hh567368.aspx) (as required by PyStan on Windows)
* clang, which will basically always be a superset of the above two and so won't hold us back.

# Language Features
### [C++11 features](http://blog.smartbear.com/c-plus-plus/the-biggest-changes-in-c11-and-why-you-should-care/)
* everything except for Expression SFINAE
### C++14 features
* auto and decltype(auto) return types
* generic lambdas


There is some discussion here: http://discourse.mc-stan.org/t/moving-to-c-11-again/75/21
