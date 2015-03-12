This page is for notes about allowing C++11 in Stan.

#### Stan C++ API, CmdStan,

No worries.  These all just use C++ directly.


#### RStan

R is moving to GCC 4.9.2:

http://developer.r-project.org/blosxom.cgi/R-devel/NEWS/2015/02/20#n2015-02-20

Avi Avrham reported:  [This thread](https://stat.ethz.ch/pipermail/r-devel/2015-March/070785.html) on r-devel is a bit long but the latest Rtools is based on GCC 4.9.2, and R builds properly, so when 3.2 is released in April, you can probably start taking advantage of cool C++11 stuffs.

I doubt that the default binary of R will be built with std=c++11 or gnu++11, but most people running stan should be able to compile R as they need to for stand anyway, no?

#### PyStan

Newer versions of Python will not have a problem (e.g., Python 3.x) with C++11 extensions. The entire scientific computing "stack" in Python (numpy, etc) is on Python 3 these days so as long as there's a deprecation plan with nice visible, public warnings, I think Stan should go ahead and use C++11.

Insofar as Windows support is concerned, it's important that the C++11 features used are supported by *Visual Studio 2010*.  

https://wiki.apache.org/stdcxx/C++0xCompilerSupport


####  MatlabStan, Stan.jl

No worries.  These call CmdStan in a separate process.