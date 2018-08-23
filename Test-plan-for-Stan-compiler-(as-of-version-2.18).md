# Overview

This document describes the current set of tests for the Stan compiler.  The compiler consists of:
- a preprocessor which assembles one or more files of .stan code into a single input file
- a single-pass parser which reads the input file and translates it to a c++ object representing the AST (abstract syntax tree) in the case that the input is a syntactically correct Stan program, or fails to parse with an error message if the input file is incorrect.
- a generator which translates the AST object into a single c++ file.

# Tests

## Unit tests

The directory `src/test/unit/lang` contains a set of unit tests for the Stan compiler.

### Preprocessor unit tests

### AST unit tests

### Parser unit tests

### Generator unit tests

## Integration tests

makefile test: `make test/integration/compile_models`
- Tests that all Stan programs under `src/test/test-models/good` be compiled, linked, and loaded into a runtime executable.

## End-to-end tests

https://github.com/stan-dev/performance-tests-cmdstan This is a system which takes a set of known models and inputs, compiles and runs the models and then checks that the outputs obtained are within epsilon of the outputs of the current release(?)   The set of models comes from repo https://github.com/stan-dev/stat_comp_benchmarks.

# Discussion

Some features of the Stan language are only lightly tested, or tested mainly by unit tests on the AST and parser.  We need more end-to-end testing, especially w/r/t features that are easy to screw up, e.g., indexing.
