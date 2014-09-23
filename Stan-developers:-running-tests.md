This document describes how to run tests within the Stan repository.

1. Prerequisites.
2. The Python script, runTests.py.


### 1. Prerequisites

1. Required programs:
  - C++ compiler: g++ or clang++
  - Python: 2.7, 2.8, or 3.0
  - Make, find, grep

2. Download Stan. (This can be within an interface like CmdStan.) Change directory to the Stan directory.

3. Add makefile customizations to the file `make/local`. Things that you may want to include:
  - `CC` is the C++ compiler flag. Example: `CC=clang++`
  - `O` is the optimization flag. Example: `O=0`
  
  **Shortcut** If you want to add `CC=clang++` and `O=0` to `make/local` type:
```
> echo "CC=clang++" >> make/local
> echo "O=0" >> make/local
> cat make/local
```

4. Change access permissions to the Python script `runTests.py`. From the command line type:
   
   `> chmod +x runTests.Py`

   This only needs to be done once.


#### Git users

**Note:** This only needs to be done once.

If your Stan directory was cloned, either stand-alone or through an interface, we will force git to ignore changes made to `make/local`. From the command line type:
```
> git update-index --assume-unchanged make/local
```
  
Without the above change, git will show `make/local` as changed when running `git status`.


### 2. The Python script, runTests.py

The `runTests.py` Python script is responsible for

1. building the test executables specified using make
2. running the tests specified

Tests executables will be rebuilt if any changes are made to dependencies of the executable.

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
