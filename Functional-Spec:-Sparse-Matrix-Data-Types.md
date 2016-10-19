Everyone agrees that it'd be nice to have sparse matrices and vectors in Stan that would be supported for all the operations for which dense matrices are supported.  It's not so straightforward.

## How to Construct a Sparse Matrix

#### Coordinate List Notation

This is what Eigen uses internally.  A sparse matrix is constructed from:

* a size specification number of rows and number of colums `(int rows, int cols)`

* a sequence of row index, column index, and value, `(int[] row_idx, int[] col_idx, real[] value)`.

#### Compressed Sparse Row (CSR) Notation

* a sequence of row starts `(int[])`

* a sequence of column index and value `(int[], real[])`

This will take up less space than the coordinate list notation, but is harder to understand and code.

#### Sparse Vectors and Row Vectors

We'll also want `sparse_vector` and `sparse_row_vector` types, which should follow the sparse matrix design in the obvious way.

## Declaring a Sparse Matrix in Stan

#### Data

Stan input base class `var_context` requires a data shape and size specification to read a structured type like a covariance matrix or simplex.   For a sparse matrix, a full specification indicates the size of the matrix (rows and columns) and a specifiation of which values are stored (all other entries are implicitly zero).  In Stan,

```
sparse_matrix[M, N, mm, nn] a;
```

where the number of non-zero entries is `K` and the types of the above are:

```
int<lower=0> M;
int<lower=0> N;
int<lower=1, upper=M> mm[K];
int<lower=1, upper=N> nn[K];
```

Stan knows the sizes of its containers, so `K` can be calculated from `mm` and `nn` when they are compared for consistency.

Sparse vectors are the same only with a single size and a single index array.

#### Parameters

The same specification would also work for parameters, which must always have the same shape and are not assignable.

#### Transformed Data, Transformed Parameters, and Generated Quantities

These could arguably be defined dynamically and declared simply with

```
sparse_matrix[M, N] a;
```

where `M` and `N` are the numbers of rows and columns respectively.  

If Eigen's sparse matrices are dynamically assignable, this might be relatively efficient;  otherwise, we'll have to generate a whole new matrix if we try to assign to a value that's not already there.

The major issue here is that transformed parameters and generated quantities must always have the same full specification each iteration to be compatible with I/O and posterior analysis infrastructure, which assumes a draw for each parameter each iteration.  The tension is that if we define one of these variables as a function of other variables, it's hard to predict the resulting shape, but if we don't, we won't be able to validate we're getting the same variables each iteration.

#### Local Variables

These can be more flexible because they don't need to be the same shape each iteration.

## Indirect Data Type

An alternative is to not have sparse matrices as a declared type, but have them only as a result type.  That is, we could do something like this:


```
parameters {
  real theta_raw[K];
...
model {
  sparse_matrix[M, N] theta = to_sparse_matrix(M, N, mm, nn, theta_raw);
  ...
```

This makes it hard on the user to code the model input in some cases and hard to interpret the output because they'll just get theta_raw.  

It will be much easier to code other than that there will be three new data types that are not like the others in that they can only be declared as local variables.

## Sparse Matrix Operations

#### Sizing

We need operations:

```
int rows(sparse_matrix a);
int cols(sparse_matrix a);
int nonzero_size(sparse_matrix a);
real operator[](sparse_matrix a, int m, int n);
```

We probably also want

```
sparse_vector operator[](sparse_matrix a, int m);
```

#### Indexing and Assigning

We of course want to be able to index and get values out, as in `a[3, 2]`.  This should return 0 if the value isn't defined.

The big question is whether a sparse matrix should be allowed to be an lvalue---that is, something that can be assigned to.  And if so, will it only be assignable at coordinates at which it is already defined on construction?

#### Matrix Arithmetic

Yes, please.

#### Linear Algebra

Yes, please.

#### Mixed Operations

How about multiply a sparse matrix by a dense vector?  We already use that operation in RStanArm and it's Stan's one supported sparse matrix operation.  I don't think it's something Eigen supports other than by promoting the sparse vector up to dense.
