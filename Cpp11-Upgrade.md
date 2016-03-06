This page is for notes about allowing C++11 in Stan.

#### Stan C++ API, CmdStan,

No worries (other than C++11 has not been tested much with Stan).  These all just use C++ directly.

#### RStan

R is moving to GCC 4.9 on April 13, 2016: https://github.com/rwinlib/r-base#readme

As of Rtools 3.3, the GCC is based on version 4.9.3, but there are seperate toolchains for 32-bit and 64-bit.

#### PyStan

Python will not have a problem with C++11. Previously, Windows was a concern but this is no longer the case. (Python 3.5 on Windows uses Visual C++ 2015 (also known as Visual C++ 14.0).)

####  MatlabStan, Stan.jl

No worries.  These call CmdStan in a separate process.