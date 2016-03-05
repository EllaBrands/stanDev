This page is for notes about allowing C++11 in Stan.

#### Stan C++ API, CmdStan,

No worries (other than C++11 has not been tested much with Stan).  These all just use C++ directly.

#### RStan

R is moving to GCC 4.9 on April 13, 2016: https://github.com/rwinlib/r-base#readme

As of Rtools 3.3, the GCC is based on version 4.9.3, but there are seperate toolchains for 32-bit and 64-bit.

#### PyStan

Newer versions of Python will not have a problem (e.g., Python 3.x) with C++11 extensions. The entire scientific computing "stack" in Python (numpy, etc) is on Python 3 these days so as long as there's a deprecation plan with nice visible, public warnings, I think Stan should go ahead and use C++11.

Insofar as Windows support is concerned, it's important that the C++11 features used are supported by *Visual Studio 2010*.  

https://wiki.apache.org/stdcxx/C++0xCompilerSupport

Allen said Windows would be okay if we switched to C++11.

####  MatlabStan, Stan.jl

No worries.  These call CmdStan in a separate process.