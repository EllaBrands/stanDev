The Stan repository contains functional and unit tests under the directory `src/test`.  There is a makefile for Gnu make that runs these targets, `makefile` and a helper Python script `runTests.py` which calls this makefile to build and run the targets.

1. Prerequisite tools and settings.
2. How to run tests via script `runTests.py`.

### 1. Prerequisite tools and settings.

The following programs and tools are required:
 - a C++ compiler, either g++ or clang++
 - Gnu make, version 3.8 ([http://www.gnu.org/software/make/])
 - Python:  versions 2.7 and up
 - Unix utilities `find` and `grep`

To customize Stan's build options, variables can be set in a file called `make/local`. This is the preferred way of customizing the build. You can also set it more permanently by setting variables in `~/.config/stan/make.local`.

Note: variables in `make/local` override variables in `~/.config/stan/make/local`.

**Nota bene:** these build instructions are not in the released version yet. This is for v2.18.0++. Prior to this, the build instructions are similar, but not identical.

There are a lot of make variables that can be set. In general, `CXXFLAGS_*` is for C++ compiler flags, `CPPFLAGS_*` is for C preprocessor flags, `LDFLAGS_*` is for linker flags, and `LDLBIS_*` is for libraries that need to be linked in.

These are the more common make flags that could be set:
- `CXX`: C++ compiler
- `MATH`: location of the Stan Math library.
- `CXXFLAGS_OS`: compiler flags specific to the operating system.
- `CPPFLAGS_OS`: C preprocessor flags specific to the operating system.
- `O`: optimization level. Defaults to `3`.
- `O_STANC`: optimization level for building the Stan compiler. Defaults to `0`.
- `INC_FIRST`: this is a C++ compiler option. If you need to include any headers before Math's headers, this is where to specify it. Default is empty.
- `LDFLAGS_OS`: linker flags for the operating system
- `LDLIBS_OS`: link libraries for the operating system

For additional make options, see [Building and Running Tests](https://github.com/stan-dev/math/wiki/Developer-Doc#building-and-running-tests) in the Math wiki.

The `make/local` file in the Stan repository is empty. To add `CXX=clang++` and `O=0` to `make/local` type:
```
> echo "CXX=clang++" >> make/local
> echo "O=0" >> make/local
```

### 2. The Python script, `runTests.py`

The `runTests.py` Python script is responsible for:
 1. building the test executables using Gnu make
 2. running the test executables

Test executables will be rebuilt if any changes are made to dependencies of the executable.

Running the script without any arguments brings up some help. This is the current help message:
```
> ./runTests.py
usage: ./runTests.py <path/test/dir(/files)>
or
       ./runTests.py -j<#cores> <path/test/dir(/files)>
```

Arguments:
- the first optional argument is `-j<#cores>` for building the test in parallel.  If this argument is present, it will be passed along to the <code>make</code> command and will override system defaults and `MAKEFLAGS` set in <code>make/local</code>.
- the rest of the arguments should either specify a single test file (`*_test.cpp`) or a test directory.

#### Example: running a single unit test.

To run a single unit test, specify the path of the test as an argument. Example:
```
> ./runTests.py src/test/unit/math/prim/scal/fun/abs_test.cpp
```

To run multiple unit tests at once, specify the paths of the tests separated by a space. Example:
```
> ./runTests.py src/test/unit/math/prim/scal/fun/abs_test.cpp  src/test/unit/math/prim/scal/fun/Phi_test.cpp
```

These can also take the `-j` flag for parallel builds. For example:
```
> ./runTests.py -j4 src/test/unit/math/prim/scal/fun/abs_test.cpp
```

**Note:** On some systems it may be necessary to change the access permissions to make `runTests.py` executable:
   `> chmod +x runTests.Py`
This only needs to be done once.

#### Example: running a directory of tests.

To run a directory of tests (recursive), specify the directory as an argument. Note: there is no trailing slash (`/`). Example:
```
> ./runTests.py src/test/unit/math/prim
```

To run multiple directories, separate the directories by a space. Example:
```
> ./runTests.py src/test/unit/math/prim src/test/unit/math/rev
```

To build in parallel, provide the `-j` flag with the number of cores. Example:
```
> ./runTests.py -j4 src/test/unit/math
```
