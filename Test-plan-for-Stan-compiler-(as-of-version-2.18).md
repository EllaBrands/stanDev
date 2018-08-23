# Overview

This document describes the current set of tests for the Stan compiler.  The compiler consists of:
- a preprocessor which assembles one or more files of .stan code into a single input file
- a single-pass parser which reads the input file and translates it to a c++ object representing the AST (abstract syntax tree) in the case that the input is a syntactically correct Stan program, or fails to parse with an error message if the input file is incorrect.
- a generator which translates the AST object into a single c++ file.

