### Goal

Use JSON instead of S dump format for data input files to Stan.

### Stan data

Data input files are used to hold model data and parameters.
Model data is read in by the model constructor.
Model parameters are used to initialize the sampler and optimizer.

Stan data types (section 2.2 of the Stan manual):

The primitive Stan data types are real for continuous scalar quantities and int for integer values. 
A real may be an integer or a real value or it may be one of three special values: positive infinity, negative infinity, and "not a number" which represents error conditions.  The compound data types include vector (of real values), row_vector (of real values), and matrix (of real values).  Stan supports arrays of arbitrary order of any of the basic data types or constrained basic data types.




Integer or real types may be constrained with lower bounds, upper bounds, or both. There are four constrained vector data types, simplex for unit simplexes, unit_vector for unit-length vectors, ordered for ordered vectors of scalars and positive_ordered for vectors of positive ordered scalars. There are specialized matrix data types corr_matrix and cov_matrix for correlation and covariance matrices.

### JSON



