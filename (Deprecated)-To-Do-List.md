<br />

<big>_**Warning:** This page is going away in favor of the issue tracker with lots of tags.  Please don't add anything here, but feel free to move things from here to issues (and delete what was here)._</big>

<br />

### Priorities

#### Efficiency/Scalability
* (Daniel) full multivariate probability function tests
* (Marcus, Bob, Daniel) vectorization of multivariate prob functions
* (Bob, Daniel) vectorizing univariate prob functions to matrices
    * all arrays/matrices, etc. above base type are of same dimension
* special function vectorization
* multi-threading

#### Modeling Language <a id="modeling-language"></a>
* (Bob) mixing local (and other?) variables into statements instead of requiring them to be at the top of blocks [from https://github.com/stan-dev/stan/issues/5]
* (Bob) unsized variable declaration that can then be read in
    * this has also been requested for data so as not to have to specify its size in the data file
    * could later assign to a local, or if it's an array, even allow push_back(), etc.
    * this could mess with output if parameters or transformed parameters or generated quantities changed sizes, so probably limit to data and transformed data and locals
    * [from https://github.com/stan-dev/stan/issues/5]
* (Bob) break and continue control operations
* (Bob) new types: lower triangular (+/- strict), diagonal matrix, symmetric matrix, Cholesky factor of pos-def matrix
* ragged arrays
* sparse vectors, matrices, arrays
* subroutines in modeling language
* initialization block in model w. randomization
* (Bob) user-specified language extensions (plug in facility to declare, compile, link)
* implicit functions with derivatives (for diff eqs?) 
* user-callable transforms with Jacobian lp__ adjustment
* unconstrained parameterizations of prob functions
* (Bob) syntactic improvements in language (e.g., multiple declares, compound declare and assign)


### C++ API <a id="c++-api"></a>
* Multivariate normal cdfs (and ccdfs?).  Notes from noahmotion (GitHub ID) to github:
For what it's worth, Alan Genz has (from what I can tell) good Fortran code for calculating multivariate normal integrals (I don't know how easy it would be to adapt Fortran code for use in Stan, but I thought I'd at least attempt to be as helpful as I could when asking for a feature like this).  He also has an R package called mvtnorm.  Tom Wickens' gives the partial derivatives for the bivariate Gaussian CDF in his 1992 J. of Math. Psych. paper (the broader point of which is to describe the Newton-Raphson algorithm for maximum likelihood estimation of the parameters in a multidimensional signal detection/probit model).  Citations:
    * http://www.math.wsu.edu/faculty/genz/software/fort77/mvndstpack.f
    * http://www.math.wsu.edu/faculty/genz/software/fort77/mvnpack.f
    * http://www.math.wsu.edu/faculty/genz/papers/mvn.pdf
    * http://www.math.wsu.edu/faculty/genz/papers/bvnt.pdf
    * http://www.sciencedirect.com/science/article/pii/0022249692900378
* add "multi_logit_log(int|vec,vec)" with derivs so that
    ```
    y ~ multi_logit(beta,x);
    ```
    behaves as:
    ```
    y ~ categorical(softmax(dot_product(beta,x)));
    ```
    c.f. MCMCmln in MCMCpack for speed comparison using their data set
* (from Frederic Bois): put simulated annealing or tempering in stan...

* add STAN_NO_DEBUG option to turn off bounds checking
* Speed test pass-by-const-ref and pass-by-val for stan::agrad::var
* stepwise debugger for Stan
    * print adaptation params
    * print gradients
    * just a mode on command to print more out per sample?
    * ideally link runtime errors back to lines of Stan code

### Modeling
* repeat <statement> until <cond>;
* for (<statement>; <cond>; <statement>) statement;
* for-each statement
    * foreach (<var> : <list-expression>) <statement>;
* multi-returns and array literals
    * (a,b,c) = foo();  // function returns array
    * (m,n,p) = (1,2,3);   // (1,2,3) is array literal
    * ((1,2),(3,4)) // matrix literal
* matrix and vector literals
    * The syntax is pretty simple modulo size conformance checking:
    ```
    matrix_expression ::= '[' matrix_row (';' matrix_row)* ']'
    matrix_row ::= expression*
    ```
    * The idea is that the expressions are laid out in a row and a ';' indicates to start a new row. The basic cases are as follows:
    *  [1 2 3]    : row_vector[3]
    *  [1 2 3; 4 5 6]  : matrix[2,3]
    * Use transpose to create vectors:
        * [1 2 3]'   : vector[3]
    * And allow more complex things in a matrix_row in the BNF,  such as a sequence of vectors:
    ```
    [[1 2 3]' [4 5 6]']  : matrix[3,2]
    ```
    * In general, we could allow matrices, too:
    ```
    [[1 2; 3 4] [5 6; 7 8]] : matrix[4,2]
    ```
This would keep nesting in the obvious ways.
* Array literals distinguished from matrices and vectors.  e.g,
```
  array ::= 'a' '(' expression* ')'
```
Examples are:
```
a(1,2,3)   :  int[3]
a(1.0,2,3)  :  real[3]
a(a(1,2,3), a(4,5,6)) : int[2,3]
a([1 2 3], [4 5 6]) : row_vector[3][2]
```
But it brings up type inference issues, because a(1,2,3) is int[3], not real[3].   We could  add primitive 1D-array syntax such as:
```
'ints' '(' int_expression* ')'
'reals' '(' real_expression* ')'
```
* multiple declarations
    * e.g., double a, b, c[N];
* declare and assign
```
    double b[5];
    double a[5] <- b; // odd, eh?
    simplex(5)[2] c <- ... // wouldn't be obvious to support
```
* tuple types for multiple returns (U,S,V) <- svd(A)
    * different than array literals as may have different typed components
* introduce transforms into the Stan language that have the same effect as the declarations, e.g.
```
    parameters {  real x; }
    transformed_parameters { real(0,1) y; y <- lub(y,0,1,lp__); }
```
* user extensions
    * declare file paths to #include
    * declare namespace usings
    * declare function sigs for parser 
    * if these happen up front, can be used for parsing rest of model
* compound op-assigns in StanGM
    *   +<-, *<-, etc.  (ugly, ugly, ugly syntax; cf., +=, *=, etc.)
* power operations
    * matrix power:    A^n
    * matrix elementwise power:  A.^x
    * regular old real power: x^y
* allow normal_log(y|0,1) as synonym for normal_log(y,0,1)
* add a parameter initialization block
    * fill in values not given randomly or report error?
* add randomization around initialization 
* lower triangular matrix type
    * strict lower triangular matrix
    * covariance matrix cholesky factor matrix
    * diagonal matrix
    * symmetric matrix [need to treat like others and just test at end]
* positive-definiteness-preserving operations
* unit matrix function unit_matrix(K)
    * return var or just double?  
    * users shouldn't use this to assign to transformed param
* random generation
    * generated quantities block
    * use sampling notation  y ~ normal(0,1); to set y to normal_random(0,1) draw
    * transformed data block
    * could also use sampling notation
    * issue of whether to share across chains or not
    * transformed parameter block, possible, but can't use sampling notation
    * need to push RNG through to whatever calls necessary
    * if we add to models, etc., should derivatives just be the same as basic so that
    ```
    y[n] <- random_normal(mu,sigma);
    ```
gets same grad as y[n] ~ normal(mu,sigma);
* truncations need to be vectorized, too!

### Command-Line <a id="command-line"></a>

### Testing <a id="testing"></a>
* agrad distribution tests misuse the autodif stack and may result in unexpected behavior now. Need to fix.
* build unit tests from the BUGS models that just check the gradients are right (this'll also cover parsing and being able to fire up the models)
* auto testing of sampler
    * Cook/Gelman/Rubin-style interval tests
    * needs effective sample size calc to bound intervals
    * needs simulated parameters to test estimators
    * push ESS "measurement error" through the interval tests
* more matrix tests (for all ops w. gradients)
* benchmarks
    * agrad vs. RAD/Sacado, CppAD, ... ?
    * stan vs. MCMCpack, R, BUGS, JAGS, OpenBUGS, ...
    * profile running models w. various operations
* complete math robustness and limits review
    * want domain boundaries to be right (usually -inf, 0 and inf, etc.)
* speed measurement
    * to converge
    * time per effective sample once converged
    * time to 4 chains at 25 ESS/chain, rHat < 1.1 for all parameters theta[k] and theta[k]*theta[k]
    * discarding first half of samples too many if fast to converge
* Speed vs. JAGS
    * run JAGS compiled with g++ -O3
    * Look at models where Stan performs well; e.g.  logistic or linear regression w. Cauchy and Normal prior
    * varying model shapes (step up exponentially)
        * number of data items
        * number of parameters
        * correlation of parameters (multi-variate Cauchy or Normal)
        * interaction of params (? need this with correlation ?)
        * conjugate vs. non-conjugate priors (well and misspecified -- more stats than speed issue)
    * Single threaded
    * Run 4 chains (each of same length) of each system (Stan and JAGS)
        * diffuse initializations (hand generate for each)
        * until convergence by split R-hat (all params under 1.05)
        * total (across all chains) ESS is > 100 (and < 150 or something?) (e.g., 25/25/25/25, or 15/35/20/30)
        * report time in effective samples/second for min ESS parameter
    * Report multiple runs of each (10? 100?)
        * histograms/densities or mean/sd of multiple runs
        * how much will it vary by seed
    * Throw away first half of each run?
        * to start, until split convergence/sample speeds
    * Ideally, measure:
        * time to converge to equilibrium
        * ESS rate at equilibrium

