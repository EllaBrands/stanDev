## Proposal

Use JSON instead of S dump format for data input files to Stan.

The model file contains the data declarations.
Data definitions can be suppled in a separate data file. 
Data input files are used for both model data and parameters.
Model data is read in by the model constructor.
Model parameters are used to initialize the sampler and optimizer.

The current data file format is a series of R dump statments which assign values to variables.
To use JSON notation instead, instead of assignment statements, a data definition would be expressed as a name value pair consisting of the variable name and a value of the proper type.  To make the data file a well formed JSON object, the data defitions are separated by commas and the entire set of definitions are enclosed by curly braces.

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

### Stan data types

To summarize: (taken from section "Stan Data Types" in the Stan manual)
* The primitive Stan data types are real for continuous scalar quantities and int for integer values.  A real may be an integer or a real value or it may be one of three special values: positive infinity, negative infinity, and "not a number" which represents error conditions. 
* In order to represent the real values  "NaN" and positive an negative infinity in JSON, we'd need to use strings, e.g., "NaN", "+inf", "-inf"
* Integer or real types may be constrained with lower bounds, upper bounds, or both.
* The compound data types include vector (of real values), row_vector (of real values), and matrix (of real values).
* Stan supports arrays of arbitrary order of any of the basic data types or constrained basic data types.  * There are four constrained vector data types, simplex for unit simplexes, unit_vector for unit-length vectors, ordered for ordered vectors of scalars and positive_ordered for vectors of positive ordered scalars. There are specialized matrix data types corr_matrix and cov_matrix for correlation and covariance matrices.
* Stan supports arrays of arbitrary order of any of the basic data types or constrained basic data types.

### Variable declarations

Stan data and parameters are declaired in the model file.  See Stan reference manual section "Data Types and Variable Declarations."  Examples:
* primitive int: `int i;`
* primitive real: `real a;`
* 1-dimensional array of reals:  `real a[5];`
* 1-dimensional vector:  `vector[5] a;`  (all vectors are vectors of reals).
* 1-dimensional row_vector: `row_vector[5] a;`
* 2-dimensional matrix: `matrix[2,3] b;` (all matrices are matrices of reals).
* 2-dimensional array of reals: `real b[2,3];`
* array of vectors: `vector[3] b[2];`
* array of row vector: `row_vector[2] b[3];`

### Stan variable definitions as JSON name : value pairs

Data declarations, corresponding JSON data definitions:
* primitive int: `int i;`  JSON `"i" : 17`
* primitive real: `real a;` JSON `"a" : 17` `"a" : 17.2`<br>
for NaN, +inf, -inf:  JSON `"a" : "NaN"` `"a" : "+inf"`
* 1-dimensional array of reals:  `real a[5];`  JSON `"a" : [ 1, 2, 3.3, 4.0, 5 ]`<br>
because JSON doesn't require array elements to all be of the same type, the string representations for NaN and the infinities are also syntactically valid.
* 1-dimensional vector:  `real a[5];`  JSON  `"a" : [ 1, 2, 3.3, 4.0, 5 ]`
* 1-dimensional row_vector: `row_vector[5] a;` JSON  `"a" : [ 1, 2, 3.3, 4.0, 5 ]`
* 2-dimensional matrix: `matrix[2,3] b;` JSON `"b" : [ [ 1, 2, 3 ] , [ 4, 5, 6 ] ]`<br>
(even though Stan matrices are stored in column order).
* 2-dimensional array of reals: `real b[2,3];` JSON `"b" : [ [ 1, 2, 3] , [ 4, 5, 6 ] ]`
* higher dimensional arrays are possible, where dimensions are ordered first to last, which generalizes the row-major order for the above example.  E.g., `real c[2,3,2]` (two rows, three columns, 2 shelves) would be declared in JSON as `"c" : [ [ [ 1, 2 ] , [ 3, 4 ] , [ 5, 6 ] ] , [ [ 7, 8 ] , [ 9 , 10 ] , [ 11, 12 ] ] ]`
* array of vectors: `vector[3] b[2];` JSON `"b" : [ [ 1, 2, 3 ] , [ 4, 5, 6 ] ]`
* array of row vector: `row_vector[2] b[3];` JSON `"b" : [ [ 1, 2 ], [ 3, 4 ], [ 5, 6 ] ]`

A data file would be a single object consisting of multiple name-value pairs.<br>
`{ "a" : 17, "b" : [ 1, 1.1, -9e5 ] }`<br>
`{ "N" : 10, "y" : [ 0, 1, 0, 0, 0, 0, 0, 0, 0, 1 ] }`


