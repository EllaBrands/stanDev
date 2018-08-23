# Overview

This document describes the current set of tests for the Stan compiler.  The compiler consists of:
- a preprocessor which assembles one or more files of .stan code into a single input file
- a single-pass parser which reads the input file and translates it to a c++ object representing the AST (abstract syntax tree) in the case that the input is a syntactically correct Stan program, or fails to parse with an error message if the input file is incorrect.
- a generator which translates the AST object into a single c++ file.

# Tests

## Unit tests

The directory `src/test/unit/lang` contains a set of unit tests for the Stan compiler.  Features are mainly tested in isolation; some tests may test the outcome of a chain of operations.

### Preprocessor unit tests

`src/test/unit/lang/io/program_reader_test.cpp` - extensively tests pre-processor, test models in `stc/test/test-models/good/included`

### AST unit tests

- `src/test/unit/lang/ast/type` - API level tests for all variable types in the Stan language
- `src/test/unit/lang/ast/node` - tests for all variable declarations in the Stan language, extensive tests on indexing nodes, some AST nodes are not fully tested.
- `src/test/unit/lang/ast/fun`  - some tests for particularly complex functions
- `src/test/unit/lang/ast/sigs` - tests function signatures

### Parser unit tests

`src/test/unit/lang/parser` - test files have been added as new features have been added to the language.

### Generator unit tests

`src/test/unit/lang/generator` - some tests for specific language constructs.

There are a set of brittle tests which take a minimal Stan program, e.g., single block, single variable declaration, run it through the compiler, and then check that the generated c++ code is _exactly_ as expected, using simple string matching.

## Integration tests

makefile test: `make test/integration/compile_models`
- Tests that all Stan programs under `src/test/test-models/good` be compiled, linked, and loaded into a runtime executable.

Some unit tests require instantiating and running a Stan model, e.g. services and variational tests.

## End-to-end tests

https://github.com/stan-dev/performance-tests-cmdstan This is a system which takes a set of known models and inputs, compiles and runs the models and then checks that the outputs obtained are within epsilon of the outputs of the current release(?)   The set of models comes from repo https://github.com/stan-dev/stat_comp_benchmarks.

# Discussion

Some features of the Stan language are only lightly tested, or tested mainly by unit tests on the AST and parser.  We need more end-to-end testing, especially w/r/t features that are easy to screw up, e.g., indexing.
