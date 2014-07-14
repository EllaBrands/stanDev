[CMake](http://www.cmake.org) is a metabuild system which, when run, produces a build environment for a project using a chosen native toolchain.  The most common toolchain is Make, but XCode, MS Visual Studio and others are supported.  Stan is currently in the process of converting from a purely Make-based build system to CMake.  This is being done in the ```feature/cmake``` branches of Stan and CmdStan and progress is being tracked in [issue #628](https://github.com/stan-dev/stan/issues/628).

This page describes the basic process for using CMake to build and test Stan.  

## Quick Start

CMake is is designed to support out-of-tree builds and that is the recommended way to use it.  This means that all build configuration files, intermediate files, binary object files and other build output are placed under a different directory than the source directory.  The following commands should work to setup and build CmdStan:

```
mkdir build
cd build
cmake ..
make
```


## Customizing the Build

Still to be written:
* Compiler configuration
* Build type
* Building with system libraries
* Build options (docs, testing, etc)

## Advanced Build Options

Still to be written:
* MinGW builds
* Other generators (XCode, MS Visual Studio)



