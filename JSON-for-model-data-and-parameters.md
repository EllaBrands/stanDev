## Goal

Use JSON instead of S dump format for data input files to Stan.

##  JSON

JSON is a data interchange notation, defined by an [ECMA standard](http://www.ecma-international.org/publications/files/ECMA-ST/ECMA-404.pdf).  JSON data must in a Unicode format.  JSON data is a series of structural tokens, literal tokens, and values.
* Structural tokens are the left and right curly bracket, left and right square bracket, the semicolon, and the comma.  `{}{}:,`
* Literal tokens must always be in lowercase.  There are three literal tokens: `true false null`
* A primitive value is a single token which is either a literal, a string, or a number.
* All numbers are decimal numbers.  Scientific notation is allowed.  The concepts of positive and negative infinity as well as "not a number" cannot be expressed as numbers in JSON.  
* A string consists of zero or more Unicode characters or control sequences enclosed in double quotes.  A backslash is used to escape the double quote character as well as the backslash itself.
* A name-value pair consists of a string followed by a colon followed by a value, either primitive or compound.
* A JSON object is a comma-separated series of zero or more name-value pairs enclosed in curly brackets.  Each name-value pair is a member of the object.  Membership is unordered.  Member names are not required to be unique.
* A JSON array is an ordered, comma-separated list of zero or more JSON values enclosed in square brackets.  The elements of an array can be of any type.   

JSON examples:
* literals:  `true` `false` `null`
* numbers: `17` `17.2`  `-17.2` `-17.2e8` `17.2e-8`
* strings: `"foo"`
* objects: `{}` ` "foo": null }` `{ "bar" : 17, "baz" : [ 14, 15, 16.6]  }`
* arrays: `[]` `[ 1 ]` `[ "a" , "b", true ]`

##  Stan data

The model file contains the data declarations.
Data definitions can be suppled in a separate data file. 
Data input files are used for both model data and parameters.
Model data is read in by the model constructor.
Model parameters are used to initialize the sampler and optimizer.

The current data file format is a series of R dump statments which assign values to variables.
To use JSON notation instead, instead of assignment statements, a data definition would be expressed as a name value pair consisting of the variable name and a value of the proper type.

### Stan data types

From section "Stan Data Types" in the Stan manual:
* The primitive Stan data types are real for continuous scalar quantities and int for integer values.  A real may be an integer or a real value or it may be one of three special values: positive infinity, negative infinity, and "not a number" which represents error conditions. Integer or real types may be constrained with lower bounds, upper bounds, or both.
* The compound data types include vector (of real values), row_vector (of real values), and matrix (of real values).  Stan supports arrays of arbitrary order of any of the basic data types or constrained basic data types.  There are four constrained vector data types, simplex for unit simplexes, unit_vector for unit-length vectors, ordered for ordered vectors of scalars and positive_ordered for vectors of positive ordered scalars. There are specialized matrix data types corr_matrix and cov_matrix for correlation and covariance matrices.
* Stan supports arrays of arbitrary order of any of the basic data types or constrained basic data types.

### Variable declarations

Stan data and parameters are declaired in the model file.  See Stan reference manual section "Data Types and Variable Declarations."  Examples:
* primitive int: `int i;`
* primitive real: `real a;`
* 1-dimensional array of reals:  `real a[5];`
* 1-dimensional vector:  `real a[5];`  (all vectors are vectors of reals).
* 1-dimensional row_vector: `row_vector[5] a;`
* 2-dimensional matrix: `matrix[6,7] b;` (all matrices are matrices of reals).
* 2-dimensional array of reals: ``real b[6,7];``
* array of vectors: `vector[7] b[6];`
* array of row vetor: `row_vector[7] b[6];`

### Variable definitions

Data definitions can be suppled in a separate data file.  The current data file format is a series of R dump statments which assign values to variables.
To use JSON notation instead, instead of assignment statements, a data definition would be expressed as a name value pair consisting of the variable name and a value of the proper type.

The data file holds variable definitions for both data and parameters in the form of a series of assignments of values to variables.  Stan reads in the definitions of declared variables.
