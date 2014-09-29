The Stan repository contains functional and unit tests under the directory `stan/src/test`.  There is a makefile for Gnu make that runs these targets, `stan/makefile` and a helper Python script `stan/runTests.py` which calls this makefile to build and run the targets.

1. Prerequisite tools and settings.
2. How to run tests via script `runTests.py`.

### 1. Prerequisite tools and settings.

The following programs and tools are required:
 - a C++ compiler, either g++ or clang++
 - Gnu make, version 3.8 ([http://www.gnu.org/software/make/])
 - Python:  versions 2.7 and up
 - Unix utilities `find` and `grep`

The Stan `makefile` uses helper files in the directory `stan/make`.   The helper file `stan/make/local` is used to customize makefile options.  Typical customizations are setting the compiler option flags:
  - `CC` specifies the name of the C++ compiler. Example: `CC=clang++`
  - `O` specifies the optimization level. `0` for least code optimization, `3` for greatest amount of code optimization. Example: `O=0`

The `stan/make/local` file in the Stan repository is empty. To add `CC=clang++` and `O=0` to `make/local` type:
```
> echo "CC=clang++" >> make/local
> echo "O=0" >> make/local
> cat make/local
```
**Git users** If your Stan directory was cloned, either stand-alone or through an interface, you should tell git to ignore changes made to `make/local`. From the command line type:
```
> git update-index --assume-unchanged make/local
```
Without the above change, git will show `make/local` as changed when running `git status`.

This only needs to be done once.


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
- the first optional argument is `-j<#cores>` for building the test in parallel. (This cannot be specified within `make/local`)
- the rest of the arguments should either specify a single test file (`*_test.cpp`) or a test directory. 

#### Example: running a single unit test.

To run a single unit test, specify the path of the test as an argument. Example:
```
> ./runTests.py src/test/unit/math/functions/abs_test.cpp
```

To run multiple unit tests at once, specify the paths of the tests separated by a space. Example:
```
> ./runTests.py src/test/unit/math/functions/abs_test.cpp src/test/unit/math/functions/Phi_test.cpp 
```

These can also take the `-j` flag for parallel builds. For example:
```
> ./runTests.py -j4 src/test/unit/math/functions/abs_test.cpp
```

**Note:** On some systems it may be necessary to change the access permissions to make `runTests.py` executable:   
   `> chmod +x runTests.Py`
This only needs to be done once.

#### Example: running a directory of tests.

To run a directory of tests (recursive), specify the directory as an argument. Note: there is no trailing slash (`/`). Example:
```
> ./runTests.py src/test/unit/math/functions
```

To run multiple directories, separate the directories by a space. Example:
```
> ./runTests.py src/test/unit/math/functions src/test/unit/math/matrix
```

To build in parallel, provide the `-j` flag with the number of cores. Example:
```
> ./runTests.py -j4 src/test/unit/math
```