This page is for notes about allowing C++11 in Stan.

#### Stan C++ API, CmdStan,

No worries.  These all just use C++ directly.


#### RStan

R is moving to GCC 4.9.2:

http://developer.r-project.org/blosxom.cgi/R-devel/NEWS/2015/02/20#n2015-02-20

Avi Avrham reported:  [This thread](https://stat.ethz.ch/pipermail/r-devel/2015-March/070785.html) on r-devel is a bit long but the latest Rtools is based on GCC 4.9.2, and R builds properly, so when 3.2 is released in April, you can probably start taking advantage of cool C++11 stuffs.

I doubt that the default binary of R will be built with std=c++11 or gnu++11, but most people running stan should be able to compile R as they need to for stand anyway, no?

Update as of 19 March 2015:

The gcc 4.9.2 toolchain that is currently in Rtools33 has too many incompatibilities with existing code, so we won't be using it in the R 3.2.0 build.  I will soon be uploading to CRAN a new version of Rtools33 that is very similar to Rtools32, containing gcc 4.6.3.

We are continuing to work on the new toolchain, and hope to have it ready before R 3.2.1 is released.

The known problems are as follows:

* C++ code should not call Rf_error(), as it uses longjmp, and the behaviour of longjmp is undefined in C++ when destructors need to be called.  However, a number of packages do call Rf_error, and in gcc 4.6.3, they get away with it.  In our candidate 4.9.2 build, they crashed.  If we can't work around this, I'll suggest that we test for the presence of Rf_error in C++ code, and start issuing warnings or errors when it is seen.  But before we do that, we need a solid replacement.

* There are some other crashes that appear to be unrelated, also with C++ code.

* There are some subtle differences in arithmetic that result in tests failing.  These may be due to bugs in MinGW-w64 code, or may be unavoidable.
</blockquote>

#### PyStan

Newer versions of Python will not have a problem (e.g., Python 3.x) with C++11 extensions. The entire scientific computing "stack" in Python (numpy, etc) is on Python 3 these days so as long as there's a deprecation plan with nice visible, public warnings, I think Stan should go ahead and use C++11.

Insofar as Windows support is concerned, it's important that the C++11 features used are supported by *Visual Studio 2010*.  

https://wiki.apache.org/stdcxx/C++0xCompilerSupport


####  MatlabStan, Stan.jl

No worries.  These call CmdStan in a separate process.