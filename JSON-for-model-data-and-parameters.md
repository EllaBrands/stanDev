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
* Structural tokens are the left and right curly bracket, left and right square bracket, the semicolon, and the comma.  `{}[]:,`
* Literal tokens must always be in lowercase.  There are three literal tokens: `true false null`
* A primitive value is a single token which is either a literal, a string, or a number.
* All numbers are decimal numbers.  Scientific notation is allowed.  The concepts of positive and negative infinity as well as "not a number" cannot be expressed as numbers in JSON.  
* A string consists of zero or more Unicode characters enclosed in double quotes.  A backslash is used to escape the double quote character as well as the backslash itself. JSON allows the use of Unicode character escapes, e.g. `\uHHHH` where HHHH is the Unicode code point in hex. 
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
* Stan supports arrays of arbitrary order of any of the basic data types or constrained basic data types.
* There are constrained vector data types: `simplex` for unit simplexes, `unit_vector` for unit-length vectors, `ordered` for ordered vectors of scalars, `positive_ordered` for vectors of positive ordered scalars.  There are specialized matrix data types `corr_matrix` and `cov_matrix` for correlation and covariance matrices, and Cholesky factors of correlation matrix (`cholesky_factor_corr`) and covariance matrices (`cholesky_factor`)
* Stan supports arrays of arbitrary order of any of the basic data types or constrained basic data types.


## Stan declarations and JSON values

This section lists the Stan declarations and matching JSON specifications.  

### Value types

Not-a-number and infinite values are encoded as strings, which may be mixed with numbers; examples are given with the first set of real values.  

Values consisting only of integers may be assigned to integer types or to promoted real/vector types of the appropriate shape.

### Examples 

The examples are all unconstrained, but the same JSON data structure holds for constrained types, including the Cholesky factors.

#### primitive int
`int i;`

`"i" : 17`

#### primitive real
`real a;` 

`"a" : 17` 

`"a" : 17.2`

`"a" : "NaN"`

`"a" : "+inf"`

`"a" : "-inf"`

#### array of int
`int a[5];`

`"a" : [1, 2, 3, 4, 5];`

#### array of reals, vector, row vector
`real a[5];`  

`vector[5] a;`

`row_vector[5] a;`

`"a" : [ 1, 2, 3.3, "NaN", 5 ]`

#### 2D array of reals, array of vector, array of row vector, matrix
`real b[2, 3];`

`vector[3] b[2];`

`row_vector[3] b[2];`

`matrix[2, 3] b;`

`"b" : [ [ 11, 12, 13 ],
         [ 21, 22, 23 ] ]`

#### 3D array of reals, 2D array of vector, 2D array of row vector, array of matrix
`real b[2, 3, 4];`

`vector[4] b[2, 3];`

`row_vector[4] b[2, 3];`

`matrix[3, 4] b[2];`

`"b" : [ [ [111, 112, 113, 114],
           [121, 122, 123, 124], 
           [131, 132, 133, 134] ],
         [ [211, 212, 213, 214],
           [221, 222, 223, 224], 
           [231, 232, 233, 234] ] ]`


#### 4D and beyond

The pattern continues, with first-index first indexing.  

## JSON Data Files

A data file would be a single object consisting of multiple name-value pairs.

    { "a" : 17, "b" : [ 1, 1.1, -9e5 ] }
    { "N" : 10, "y" : [ 0, 1, 0, 0, 0, 0, 0, 0, 0, 1 ] }

