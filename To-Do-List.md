### To-Do List Organization
* Next Release Priorities
* Other High Priority Items
* C++ API                       
* Modeling Language             
* Build                         
* Testing                       
* Command-Line                  
* C++ API
* RStan
* Manual
* Web Pages
* Release Mgmt
* Models 

### For Next Release (1.3.0++)
* (Bob/Marcus/Daniel) look at add(), etc. operations for instantiation
    * already did multiply/divide in stan/math
    * now need to worry about agrad / int, etc. to make sure there's no auto promotion to agrad
    * look at add/subtract where there's broadcast function
    * need tests for int in all the relevant positions
* (everyone) remove std::cout from API
* (Daniel) fix bin/print spacing
* go through include-what-you-use output from Ben to mailing list (early May)
* sort out why log_prob_poly<>() isn't working for Metropolis with double
* redundant model_header.hpp includes --- why not just include matrix.hpp?
* int vs. size_t and error handling in indexing;  with size_t, get cast of negative int to large positive size_t, which will confuse users in error msgs
* Figure out how to return error messages safely
e.g., math/matrix/validate_std_vector_index.hpp
```
      std::stringstream ss;
      ss << "require 0 < index <= vector size" << msg;
      ss << "; found vector size=" << m.cols()
         << "; index j=" << j;
      throw std::domain_error(ss.str());
```
cf., [runtime-error copy string? (Stack Overflow)](http://stackoverflow.com/questions/10644910/does-stdruntime-error-copy-the-string-passed-in-the-constructor)
* (Daniel) fix order of include issues for model header, etc.  in new refactor (see Jiqiang's e-mail 4/19/13, 9:20 PM to stan-dev, and rest of that thread)
* fix error message in Stan when an error is encountered during initialization --- currently says "about to be Metropolis rejected";  at very least, change the message to qualify when rejection happens
* fix remaining template functions in stan/math to ensure they match int better than var matches int
* remove warning message on transform OR declare a set of function names to ignore

### Priorities

#### Contributed Code
* R compiler for regression syntax
* DIC, WAIC

#### Efficiency/Scalability
* (Daniel) replace tests with doubles
* (Daniel) full multivariate probability function tests
* (Daniel) vectorized derivatives for prob functions
* (Marcus, Bob, Daniel) vectorization of multivariate prob functions
* special function vectorization
* multi-threading
* (Peter, Bob, Michael) ensemble samplers: 
    * differential evolution, DREAM and differential evolution, 
    * Ensemble walk, stretch moves
* (Peter, Bob, Michael) higher-order auto-diff
* (Michael) RM-HMC, SoftAbs

#### Modeling Language
* (Bob) new types: lower triangular (+/- strict), diagonal matrix, symmetric matrix, Cholesky factor of pos-def matrix
* (Bob) discrete sampling
* (Peter) complementary CDF and log CDF, log CCDF, plus gamma distribution CDF
* ragged arrays
* sparse vectors, matrices, arrays
* matrix literal-like expressions a la MATLAB
* subroutines in modeling language
* initialization block in model w. randomization
* (Bob) user-specified language extensions (plug in facility to declare, compile, link)
* implicit functions with derivatives (for diff eqs?) 
* user-callable transforms with Jacobian lp__ adjustment
* unconstrained parameterizations of prob functions
* (Bob) syntactic improvements in language (e.g., multiple declares, compound declare and assign)

#### Command and Interfaces
* Python
* MATLAB
* Julia
* Stata

#### Cleanup
* improve special function and prob function behavior at limits
* (Daniel) simplify error handling by standardizing default policy
    * remove throw from all files except for src/stan/math/error_handling/raise_domain_error.hpp
    * Need unit-tests for error handling:
    * check_nonnegative
    * check_positive
    * matrix/check_corr_matrix
    * matrix/check_ordered
    * matrix/check_pos_definite
* Move src/stan/math/matrix/check_range.hpp to src/stan/math/error_handling/matrix/
* Update error handling in prob/distributions/multivariate/continuous/wishart.hpp
* Fix Errors coming from Boost Spirit Qi parser under clang++ 3.3:
    * Multiple unsequenced warnings from assigning to _pass in grammars under clang 3.3  [see: Bob's e-mail of 5/31/13]
    * Local return on iterators, for which Jeffrey Oldham filed a [Boost bug report:](https://svn.boost.org/trac/boost/ticket/8489)

#### Documentation
* intro applied Bayes book with RStan
* (Bob) C++ manual for API and auto-dif
* C++ code cleanup (API doc, privates, consts, doc, split files, long long)
* more example models, complete/improve BUGS models, vectorize BUGS models.
* more general and dynamic web site graphics

### C++ API
* B splines (not sure which version)
* For optimization, take curvature at mode to estimate a normal approximation and draw samples from it
* Remove exceptions in ctors for special function vari (no longer a bug)
    * /functions/inverse_softmax.hpp: #include <boost/throw_exception.hpp>
    * /functions/inverse_softmax.hpp throws std::invalid_argument if size of the input and output vectors differ.
    * /functions/lgamma.hpp throws domain_error if x is at pole
    * /functions/log1p.hpp includes <boost/throw_exception.hpp>
    * /functions/log1p.hpp throws std::runtime_error("foo");
    * /functions/softmax.hpp includes <boost/throw_exception.hpp>
    * /functions/softmax.hpp throws std::invalid_argument if size of the input and output vectors differ.
* log_minus_exp(x,y)
= log(exp(x) * (exp(x)/exp(x) - exp(y)/exp(x)))
= log(exp(x)) + log(1 - exp(y)/exp(x))
= x + log(1 - exp(y - x))
= x + log1m(exp(y-x)) 
or, following http://snipplr.com/view/12707/,
```
    double diff = x - y;
    diff /= 2.0;
    return (x + y) / 2.0 + log(exp(diff) - exp(-diff)); 
```
check x >> y and return x (do this in log_sum_exp, too?)
see: http://cran.r-project.org/web/packages/Rmpfr/vignettes/log1mexp-note.pdf
      for log(1-exp(x))
* Chebyshev polynomials of the first kind
    * with some kind of regression application example
    * Boost has very parameterized Chebyshev polynomials, so figure out which ones in which parameterizations we need
    * lm(a ~ poly(x,n)...) 
    * see 9 May 2013 post to stan-dev from dlakelan
* matrix normal distribution
    * see: http://en.wikipedia.org/wiki/Matrix_normal_distribution
    * useful for GeoBUGS type models
* clean up assign_to_var location & namespaces for auto-diff
    * work through use case from C++ API
    * intro namespaces stan::diff::fwd & stan::diff:rev
    * base instances in stan::math
    * fwd-mode instances in stan::diff::fwd
    * rev in diff::rev
* plain old expression print (as input, not as generated in C++)
    * clearer error messages for users
* to_vector(matrix):vector  function in row-major order
* kronecker product function
    * really needs new data type or not going to be efficient
* Add model methods:
    *  string model_name()
    * void constrained_param_names(vector<string>& v)
    * void constrained_values(vector<double>& v, const vector<double>& r, const vector<int>& i)
    * void unconstrained_param_names(vector<string>& v)
* add "multi_logit_log(int|vec,vec)" with derivs so that
```
y ~ multi_logit(beta,x);
```
behaves as:
````
y ~ categorical(softmax(dot_product(beta,x)));
````
c.f. MCMCmln in MCMCpack for speed comparison using their data set
* expose all the groovy special functions Michael/Daniel added for CDFs
* allow only some inits to be specified in a data file (var_context) and let the rest be randomly generated
* bound error in inits (and discuss in manual)
init file:  
````
    x <- structure(c(1,1,1), .Dim=c(3))
````
with model:
```
    parameters {
      vector<lower=0,upper=1e50>[3] x;
    }
    model {
      print("x = ", x);
      x ~ normal(0,1);
    }
```
problem is overflow with initial value of x being 1e35 due to huge bounds.
* fix Eigen (3.0) indexes to use size_t to match vector; see
    * http://eigen.tuxfamily.org/dox-devel/classEigen_1_1DenseBase.html#a4d4873e91be950c079f067fa97fd5c40  
    * http://eigen.tuxfamily.org/dox-devel/TopicPreprocessorDirectives.html
which says:
    * EIGEN_DEFAULT_DENSE_INDEX_TYPE - the type for column and row indices in matrices, vectors and array (DenseBase::Index). Set to std::ptrdiff_t by default.
* merge forward-mode and higher-order derivative functionals
* agrad for gamma_p function to support gamma_cdf
* (from Frederic Bois): put simulated annealing or tempering in stan...
* broadcast ./ operation for real / vector and vector / real.
* Refactor math library:
    * src/stan/math/functions/ibeta.hpp
    * all doubles for now. If I template it, conflicts with var definition.
    * Needs tests. Currently not tested: src/stan/math/matrix/...
        * stan_print.hpp
        * singular_values.hpp
        * cholesky_decompose.hpp
        * eigenvalues_sym.hpp
        * inverse.hpp
        * diag_matrix.hpp
        * transpose.hpp
        * diagonal.hpp
        * block.hpp
        * diag_post/pre_multiply.hpp
        * dot_product.hpp
        * columns_dot_product.hpp
        * log_determiant.hpp
* for vectorization, here are example signatures 
    * normal_log(scalar,scalar,scalar)
    * dirichlet_log(simplex,vector);  [can have double in vector, but not simplex]
    * multi_normal(vector,vector,cov_matrix);
    * categorical(int,simplex);
    * multinomial(vector,simplex);
* deprecate lkj_cov
    * work out general framework for deprecated functions with warning messages
    * include option for deleted functions with error messages
* How to deal with generated quantities
    * execute & print every time
    * warning message for print that it only happens for saved samples
* JSON I/O.  Check out:
    * http://www.boost.org/doc/libs/1_53_0/doc/html/property_tree/reference.html#header.boost.property_tree.json_parser_hpp 
* refactoring:
    * remove mcmc/hmc.hpp
    * merge nuts_nondiag and nuts_givenmass
    * take in something other than string on input for nutsmass
    * merge in nuts_diag if possible
* replace Eigen .sum() with stan::math::sum() for auto-dif
* remove (unsigned) long long instances in dump and error testing
* shift APIs from std::vector<T> to Eigen::Matrix<T,Dynamic,1>, etc.
* remove chainable() intermediate class and get rid of virtual function for init zero
* check out varying target acceptance rates, trying 1/2, 1/4, 3/4, 1/8, 7/8, ... and then divide and conquer
* evaluate different maximum number of steps
    * during warmup
    * during sampling
* rewrite functions in matrix.hpp:
    * elt_multiply() [fix inv_wishart_log, wishart_log]
    * subtract() [fix multi_normal_log]
    * should be able to write one templated function to allow for required uses.
* consider alternative transforms
    * ordered as logit(cumulative_sum(simplex)) [need to work through the implied geometry here]
* add lower/upper bound infinite or negative infinite values
    * need is_pos_inf<T>(x) and is_neg_inf(x) tests and we only have isinf() which is either/or
* make sure all range errors are being caught and allow them to be turned off
    * math/matrix.hpp line 164
    * STAN_NO_DEBUG property
    * turn off checking if (NDEBUG || STAN_NO_DEBUG)
    * impl from Marcus (Eigen):
    ```
    #ifndef STAN_NO_DEBUG
    #ifdef NDEBUG
    #define STAN_NO_DEBUG
    #endif
    #endif
    #ifdef STAN_NO_DEBUG 
    ...
    #endif
    ```
* make sure symmetry tests aren't too stringent for cov_matrix
* unpack Eigen/Boost and use just what we need a la RStan
    * For Boost, do: 
    ```
    mkdir /tmp/boost_1.52.0
    find ${STAN_HOME}/src/ -name \*\.\[ch]pp -exec bcp --scan --boost=${STAN_HOME}/lib/boost_1.52.0 '{}' /tmp/boost_1.52.0/ \; &> /tmp/boost_1.52.0/bcp.log 
    ```
* change 
```
step_size = epsilon +/- epsilon_pm
```
to
```
step_size = min(C, epsilon + normal(0,sigma))
```
* check that positive definiteness enforced for transformed params, transformed data
* switch error checking for agrad::var matrix ops to convert to double-based matrix for check as in vectorized univariates
* Ben's example: 
```
matrix[3,2] ~ normal(matrix[3,1], matrix[1,2])
```
this could recycle the first arg, a matrix shaped like a vector and the second arg, a matrix shaped like a row vector.

* higher-order auto-dif
    * chain rule:  http://en.wikipedia.org/wiki/Chain_rule#Higher_derivatives
    * store adj_, adj2_ and adj3_ in new class 
    * stan::agrad::var3 with matching stan::agrad::vari3, and also var2, vari2
    * chain() needs to compute them all in order
    * perhaps with chain1(), chain2(), and chain3(), with chain1() being the existing rule
* forward mode auto-dif
    * last revision 548: 
    * src/stan/agrad/ad.hpp
    * use for reasonably efficient Hessian calcs
* use expression template for value of += so that we could
    * sum_expression& operator+=(sum_expression&, const var&);
    * sum_expression(const var&);
    * var(const sum_expression&); 
    * create vectorized vari from sum_expression  would require running lp__, etc. to be sum_expression to be useful
    * without picking up signature var& operator+=(var&, const var&);
* templated functions returning var should go away in matrix
* add base class for stanc-generated models
* alternative distribution parameterizations
    * e.g., precision for (multi) normal
    * inverse_sqrt_gamma(...)
* handle covariance or correlation matrix of an AR1 process. 
    * If rho is the auto-regressive parameter, the entries of the correlation matrix are powers of rho. And the precision matrix is tridiagonal and known analytically. So, it would be inefficient to plow that through multivariate_normal_log().
* beta distro with mean plus "precision" params
    * beta_mean_log(theta|phi,kappa) = beta_log(theta|phi*kappa, (1-phi)*kappa)
    * Jacobian from:
    ```
        alpha <- phi * k
        beta  <- (1 - phi) * k
    ```
determinant = k  (det [[a,b],[c,d]] = ad - bc)
* von Mises distribution (with location and concentration params?)
    * requires modified/hyperbolic Bessel function of first kind of order 0 and 1
* pow special cases
    * inv(x) = pow(x,-1) = 1/x
    * inv_square(x) = pow(x,-2)
    * inv_sqrt(x) = pow(x,-1/2)
* signum function
```
int signum(real); 
```
* signed sqrt
```
  signed_sqrt(x) = -sqrt(-x) if x < 0
                 = sqrt(x)   if x > 0
```
* More matrix functions from Eigen:
    * singular vectors in addition to values
    * nice to have (U,S,V) = svd(A) syntax for this
    * norm()
    * squaredNorm()
    * lpNorm<1>
    * lpNorm<Eigen::Infinity>
* z-score function, normalize sample to zero sample mean, unit sample variance
* cumulative distribution functions
    * boundary conditions at limits of support for cdfs
    * should we have the basic cdfs on the log scale?
    * need for truncated distros mostly
    ```
    foo_cdf_lb_log(L|...) == log(1 - foo_cdf(L|...))
                          == foo_ccdf_log(L|...)

    foo_cdf_ub_log(U|...) == log(foo_cdf(U|...))
                          == foo_cdf_log(U|...)

    foo_cdf_lub_log(L,U|...) == log(foo_cdf(U|...) - foo_cdf(L|...))
                             == log_sum_exp(foo_cdf_log(U|...),                                 
                                            0, foo_ccdf_log(L|...))
    ````
    * to add:
        * calculation of cdf
        * two functions: 1) with templated policy, 2) with no policy,
        calling templated function with stan::math::default_policy()
        * TODO: test derivatives?
        * dox: add doxygen.
        * add to Stan Modeling Language Reference Manual function list
* matrix special products
    * quadratic_multiply(u,v) = u' * v * u
    * for square matrices,
    ```
    d/dx det(x' * A * x) = 2 det(x' * A * x) inv(x)'
    d/dx det(A) = det(A) inv(A)' = det(A) / A'
    ```
* sorting and interpolation
    * rank(v,s): number of components of v less than v[s]
    * ranked(v,s): s-th smallest component of v
    * interp_ln(e,v1,v2) from BUGS (whatever for?)
* add general mixture model
    * modeling language side
    ```
     simplex(K) lambda;
     vector(K) mu;
     vector(K) sigma;
     y ~ mixture(lambda,     // mixing
                 (mu,sigma), // param vecs
                 "normal");  // distro
    ```
    * C++ impl
    ```
    std::vector<double> log_ps;
    for (k in 1:K)
      log_ps.push_back(log lambda[k] + normal_log(y,mu[k],sigma[k]));
    return log_sum_exp(log_ps);
    ```
* use our arena-based alloc as STL allocator for auto-dif
    * already doing this for dot products
    * http://www.cplusplus.com/reference/std/memory/allocator/
    * what do we need to do for dtor to work?
    * need to take care of dynamic updates
* convert memory for vari to singleton 
    * compile time config with flag for thread-safe / not
* make sure std::less<T*> and std::swap<T*>  does right thing for our types (like var and vari), as these are used in "algorithms".
* figure out way to have vari elements that are NOT on the stack
    * useful for numeric_limits<var>
    * stack_vari ==> vari ==> chainable
    * local_vari ==> vari
    * (stack_vari has operator new; local_vari uses dtor?)
* allow Stan to read data in multiple formats (particulary CSV)
    * and allow multiple files to be specified for single run
* memcpy faster than copy ctor and operator= for var?
* remove dependence of prob functions like normal_log on agrad,  Eigen, etc.  [may not be easy or worth the effort]
* unfold logistic and linear regression auto-difs
* allow user-defined extensions
* dump format parser to handle:
    * integers with scientific notation
    * NaN, Inf, -Inf  (written just like that)
    * length one arrays read in like scalars
* make sure still possible to use header-only ad
* add scalar subtraction to match scalar addition of vectors (to distribute over values)
* Stan safe mode to compile BUGS/JAGS-compliant models
    * allows more error checking and optimization
* add STAN_NO_DEBUG option to turn off bounds checking
* add checks for matrix sizes in matrix ops
* Speed up compilation
    * More precompilation into object files.
        * Pros:  faster compiles
        * Cons:  it makes the code more difficult to manage and less flexible by requiring redundancy and preinstation of templates
    * More targeted imports.  
        * Pros:  faster compiles because less code to read
        * Cons:  defeats precompiled headers
* some kind of multi-threaded timer to monitor Stan
* discrete samplers
    * for special block, use main log_prob to evaluate conditional probs up to proportion
    * Gibbs for small outcome sets 
    * Gibbs for non-ordinal, non-count params, e.g., LDA topic samples
    * Slice for larger ones (adapt?)
    * what about mulitnomial with constraints? (metropolis?)
* more general built-in initializations
    * normal or student-t w. params
    * or even set uniform boundaries
    * e.g., uniform(-1,1), normal(0,1), t(0,2.5)
* cross-chain sharing of adaptation parameters, which should be similar across chains if all is going well
* set mass matrix for sampling parameters [& percolate to RStan/cmd]
    * need diagonal to be efficient
    * could allow general matrix
* ongoing code cleanup
    * const declarations wherever possible
    * more care with size_t, etc.
    * privatize everything possible
    * remove unnecessary dependencies/includes
    * convert for loops with size bounds to appropriate iterators
    * templatize std::vector/Eigen functions where possible
    * reduce code dup in gm/generator
* revise API doc for doxygen
* error handling for special functions
    * math and agrad
    * matrix sizes
    * how to configure std::log(), etc.?
* Watanabe's WAIC: Widely applicable information criterion
* implicit functions with derivatives (iterative solver)
* refactor finite diff derivative calcs to functor
```
  template <class F, typename S>
  bool finite_diffs(const vector<S>& x,
                    const F& f,
                    vector<S>& df_dx);
```
* Speed test pass-by-const-ref and pass-by-val for stan::agrad::var
* break out body of error handling to get isnan if/else inlined
* "non-parametric" models
    * approximate a la BUGS vol 3
    * merge reversible jump and HMC somehow?
* allow references in Stan language to make it easy to set eigenvalues + eigenvectors (or singular vals + vecs) in one call
* optimization API
    * functor-based concept design with templates 
    * easily pluggable with log prob in model for MAP 
* add adaptive slice sampling
    * will it fit into adaptive_sampler in mcmc?
    * goal to minimize number of function evals
        * step out: linear, with log step-in
        * doubling: exp, log step-in (but reject & more evals)
* adapt low-order approximation to an actual mass matrix
* online R-hat (split R-hat?) monitoring + means, etc.
    * need monitor interface like JAGS?
* elim stan/maths/matrix lib by adding implicit ctor for Matrix<var,...> based on Matrix<double,...>
* blocking updates on parameters
    * only update subset of parameters
    * specify blocks in program?
    * allow Gibbs on some variables
    * split HMC into blocks (aka batches)
    * in the limit do conditional with HMC!
* stepwise debugger for Stan
    * print adaptation params
    * print gradients
    * just a mode on command to print more out per sample?
    * ideally link runtime errors back to lines of Stan code

```
Models
======================================================================
* debug BUGS vol 1 kidney model --- sometimes hangs

* figure out what's going on with TS's model that's causing
  Eigen to throw an assert failure

* Implementing lasso-like glm and hierarchical glm in stan.


Modeling Language
======================================================================

* user-defined functions
  -- add a grammar rule so that any function in the .stan file whose name 
     ends in __ (or some other magic character combination) is parsed as a literal C++ function call
-- amend the target in the makefile so if a directory contains both foo.stan 
   and foo.hpp, then after parsing foo.stan to foo.cpp the make call is like
       $(CC) -include foo.hpp -c -o foo.o foo.cpp
   which according to the documentation of g++ and clang++ is equivalent to manually adding
       #include "foo.hpp"
   at the very top of foo.cpp. That way a user could declare / define a 
   function like bar__() in foo.hpp and use bar__() in foo.cpp that is generated from foo.stan.
-- issue is which namespace; easiest to do with global, but could
   also allow "::" in variable names iff they had the right markup, but gets clunky
   for parser and doc

* an extension of the modeling language to handle hierarchical models and data.  Something along the lines of 
      hierachical(vector y[N])[M]
  where any operation on y automatically replicates to the M groups.  This would be the first step towards   
  scaling to more data (accessing each y[N] independently of the others) as well as stochastic VB/HMC.


* allow array modifiers on types as well as variables
  -- int[4] x;
  -- int x[4];
  -- change all the syntax or allow both?

* add ragged array types
  -- data { int x[,]; } 
     for a possibly ragged array x
  -- easiest for data
     ++ buffer values until get postfix dims
  -- would also work for transformed data because sets
  -- would need a way to declare raggedness for params
     in order to read them in from array
  -- parameters { int x[ns]; }
     might be used if ns is an int array of sizes
  -- how to generalize this?
 --  int<lower=0> N;          // number of subjects
      int<lower=1> n_obs[N];   // num observations for subject n in 1:N
      cov_matrix[n_obs] c;     // cov matrices for subject n in 1:N
      *  The intent of the notation is that c[n] for n in 1:N
         is an (n_obs[n] x n_obs[n]) covariance matrix.
      * This notation lets us do a single final layer of
         raggedness and could apply anywhere there's a size.  So
            real y[n_obs];
         is a 2D ragged array structure where y[n] is a n_obs[n]-vector

* get line numbers into error messages

* compiler for R's linear model notation

* generalize vector ops to conforming matrices 
  -- need to work out whether we can do this and
     still preserve type inference;  e.g., x * x'
     may still be a matrix even if x is a row vector.

* integer subtypes
  - non_negative_int = int[1,]
  - natural = int[0,]
  - boolean = int[0,1]
  - categorical(K) = int[1,K]

* real subtypes
  - probability = real[0,1]
  - correlation = real[-1,1]
  - non_negative_real = real[0,inf]
  - non_positive_real = real[-inf,0]

* matrix slice and assign
  - index m
  - range-indexes: implicit ellision == :, a:, :b, a:b
  - e.g., matrix[M,N] m; vector(M) v;  m[,1] = v;

* conditional expressions
  - binary operators on int or double
    + bool op(int,int);
    + bool op(double,double); // allows promotion
    + usual comparisons:  ==,  !=,  <=,  <,  >=,  >
  - operators on conditions
    (!cond),  (cond && cond),  (cond || cond),  (cond)
  - ternary op: cond ? x : y

* conditional statements
  - if <cond> statement else if <cond> statement ... else statement;
  - while <cond> statement;
  - repeat <statement> until <cond>;
  - for (<statement>; <cond>; <statement>) statement;
  - Marcus's hack:
    if (a > b) {...}
    ==
    for  (i in 1:int_step(a-b)) {...}

* for-each statement
  - foreach (<var> : <list-expression>) <statement>;
  
* multi-returns and array literals
  - (a,b,c) = foo();  // function returns array
  - (m,n,p) = (1,2,3);   // (1,2,3) is array literal

* matrix and array notation:
- The syntax is pretty simple modulo size conformance
  checking.  Here's the BNF:
    matrix_expression ::= '[' matrix_row (';' matrix_row)* ']'
    matrix_row ::= expression*
  The idea is that the expressions are laid out in a row
  and a ';' indicates to start a new row.
  The basic cases are as follows:
    [1 2 3]    : row_vector[3]
    [1 2 3; 4 5 6]  : matrix[2,3]
  Use transpose to create vectors:
    [1 2 3]'   : vector[3]
  And allow more complex things in a matrix_row in the BNF, 
  such as a sequence of vectors:
    [[1 2 3]' [4 5 6]']  : matrix[3,2]
  In general, we could allow matrices, too:
    [[1 2; 3 4] [5 6; 7 8]] : matrix[4,2]
  This would keep nesting in the obvious ways.
- We need some way to call out arrays and allow higher
  dimensions, so we can't overload the same syntax.
  I like the following:
     array ::= 'a' '(' expression* ')'
  Examples are:
    a(1,2,3)   :  int[3]
    a(1.0,2,3)  :  real[3]
    a(a(1,2,3), a(4,5,6)) : int[2,3]
    a([1 2 3], [4 5 6]) : row_vector[3][2]
  But it brings up type inference issues, because
  a(1,2,3) is int[3], not real[3].   We could 
  add primitive 1D-array syntax such as:
           | 'ints' '(' int_expression* ')'
           | 'reals' '(' real_expression* ')'


* multiple declarations
  - e.g., double a, b, c[N];

* declare and assign
  - e.g., double x <- 5.0;
    double b[5];
    double a[5] <- b; // odd, eh?
    simplex(5)[2] c <- ... // wouldn't be obvious to support

* tuple types for multiple returns (U,S,V) <- svd(A)
  -- different than array literals as may have different typed
  components

* introduce transforms into the Stan language that have the same
  effect as the declarations, e.g.
    parameters {  real x; }
    transformed_parameters { real(0,1) y; y <- lub(y,0,1,lp__); }

* extensions
  - declare file paths to #include
  - declare namespace usings
  - declare function sigs for parser  
    ++ if happens up front, in time for body parsing

* ragged arrays
  - ready to use with vec<...<vec<x>...> pattern
  - not clear how to declare (follow C?)
  -- example on p. 8 of 
     http://www.jstatsoft.org/v36/c01/paper/
     involving ragged param arays for graded response model

* subroutines
  - user-defined functions in StanGM
  - user-defined densities (how much to require?)
  - interpreted conditionally based on block?

* compound op-assigns in StanGM
  +<-, *<-, etc.  (ugly, ugly, ugly syntax; cf., +=, *=, etc.)

* warnings
  - a <- foo(theta) if LHS contains variable: overwrite var
  - a ~ foo(theta)  if LHS is complex expression: need Jacobian
  - check each parameter shows up on LHS of ~ at least once

* power operations
  - matrix power:    A^n
  - matrix elementwise power:  A.^x
  - regular old real power: x^y

* allow normal_log(y|0,1) as synonym for normal_log(y,0,1)

* Allow normal_log<true> and normal_log<false> to be accessed
  directly

* add a parameter initialization block
  - fill in values not given randomly or report error?
  - needs randomization

* new types
  -- lower triangular matrix
     ++ strict lower triangular matrix
     ++ covariance matrix cholesky factor matrix
  -- diagonal matrix
  -- symmetric matrix
     [need to treat like others and just test at end]
  -- point on hypersphere
     ++ like simplex, but 2-norm is 1, rather than 1-norm
     ++ trickier for generating last point as the sign
        remains underdetermined

* positive-definiteness-preserving operations
  -- rescale strictly positive
      

* unit matrix function unit_matrix(K)
  -- return var or just double?  
  -- users shouldn't use this to assign to transformed param

* constants for integer max and min values

* random generation
  -- generated quantities block
     ++ use sampling notation  y ~ normal(0,1);
        to set y to normal_random(0,1) draw
  -- transformed data block
     ++ could also use sampling notation
     ++ issue of whether to share across chains or not
  -- transformed parameter block
     ++ possible, but can't use sampling notation
  -- need to push RNG through to whatever calls necessary
  -- if we add to models, etc., should derivatives just
     be the same as basic so that
        y[n] <- random_normal(mu,sigma);
     gets same grad as:
        y[n] ~ normal(mu,sigma);
   
* truncations need to be vectorized, too!



Build
======================================================================

* remove writes into Stan directory itself

* set up make to work from directory other than STAN_HOME
  - pass in arg?  environment var?


Testing
======================================================================

* agrad distribution tests misuse the autodif stack and may result in
  unexpected behavior now. Need to fix.
  
* build unit tests from the BUGS models that just check
  the gradients are right (this'll also cover parsing and
  being able to fire up the models)

* auto testing of sampler
  - Cook/Gelman/Rubin-style interval tests
  - needs effective sample size calc to bound intervals
  - needs simulated parameters to test estimators
  - push ESS "measurement error" through the interval tests

* more matrix tests (for all ops w. gradients)

* benchmarks
  - agrad vs. RAD/Sacado, CppAD, ... ?
  - stan vs. MCMCpack, R, BUGS, JAGS, OpenBUGS, ...

* profile running models w. various operations

* complete math robustness and limits review
  - want domain boundaries to be right (usually
    -inf, 0 and inf, etc.)

* more diagnostics during warmup of where epsilon is going

* speed measurement
  - to converge
  - time per effective sample once converged
  - time to 4 chains at 25 ESS/chain, rHat < 1.1 for all
    parameters theta[k] and theta[k]*theta[k]
  - discarding first half of samples too many if fast to converge

* Speed vs. JAGS
  -- run JAGS compiled with g++ -O3
  --  Look at models where Stan performs well
      e.g.  logistic or linear regression w. Cauchy and Normal prior
     - varying model shapes (step up exponentially)
        + number of data items
        + number of parameters
        + correlation of parameters (multi-variate Cauchy or Normal)
        + interaction of params (? need this with correlation ?)
        + conjugate vs. non-conjugate priors
           (well and misspecified -- more stats than speed issue)
  -- Single threaded
  -- Run 4 chains (each of same length) of each system (Stan and JAGS)
      - diffuse initializations (hand generate for each)
      - until convergence by split R-hat (all params under 1.05)
      - total (across all chains) ESS is > 100 (and < 150 or something?)
        (e.g., 25/25/25/25, or 15/35/20/30)
      - report time in effective samples/second for min ESS parameter
  -- Report multiple runs of each (10? 100?)
      - histograms/densities or mean/sd of multiple runs
      - how much will it vary by seed
  -- Throw away first half of each run?
      - to start, until split convergence/sample speeds
  -- Ideally, measure:
      - time to converge to equilibrium
      - ESS rate at equilibrium


RStan / PStan / MStan / Command Line
======================================================================

* Doc optimization usage.  Here's Ben's example:

----------------------------------------------------------------
# example from glm()
utils::data(anorexia, package = "MASS")
anorex.1 <- glm(Postwt ~ Prewt + Treat + Prewt,
                family = gaussian, data = anorexia)

# extract data
X <- model.matrix(anorex.1)
y <- anorexia$Postwt

# write a usual stan model
model_code <-
"
data {
  int<lower=1> K;
  int<lower=K> N;
  matrix[N,K]  X;
  vector[N]    y;
}
parameters {
  vector[K] beta;
  real<lower=0> sigma;
}
model {
  y ~ normal(X * beta, sigma);
}
"

# compile
fit <- stan(model_code = model_code, chains = 0,
            data = list(K = ncol(X), N = nrow(X), X = X, y = y))

# wrappers for log-likelihood and gradient
fn <- function(par) log_prob(fit, par)
gr <- function(par) grad_log_prob(fit, par)

# optimize
opt <- optim(c(coef(anorex.1), log_sigma = 2), fn, gr,
             method = "BFGS", control = list(fnscale = -1), hessian = TRUE)
print(opt)
----------------------------------------------------------------

* (RStan) install checklist -- especially the no spaces thing!

* (RStan) Run the examples rather than dontrun;  see Ben's e-mail
  of 2013-03-02 (March 2) titled "[stan-dev] Re: final checklist for
  1.2 release?" for instructions on caching

* (PStan) Simple JSON-based I/O interface

* (RStan) Add RStan doc to reference manual


* (cmd) more RStan like behavior
  -- add --chains=N argument to command-line to run multiple chains
     and write into one or more files (already done in RStan)
  -- new command to read .csv files into chains object and print out
     summary (related to a single command)

* (PStan) interface to Python

* (MStan) interface to MATLAB

* (R/cmd) Convergence monitoring of higher moments
  -- at least squared for variance, which being non-linear
     function, isn't same as convergence of basic param

* (R/cmd) track rejection rate by testing equality of
  samples; e.g., sum(a[1:(N-1)] == a[2:N])

* (R/cmd) restart from where command left off a la R2jags
  - this will require writing dump output or a converter
    for the CSV output to dump output
  - easiest to save unconstrained params
  - also need to save adaptation states, etc.
  - need to save seed and type of randomizer

* auto convergence monitoring -- just keep doubling (or other ratio)
  - doubling reduces total calc to at most double the end
  - easy to monitor averages and variances online using accumulators

* (R/cmd) allow configuration of target reject rate in step-size adaptation

* (cmd) write analyze command to read in output and do analysis 
  a la print in RStan

* (R/cmd) write model text into output in a comment

* (R) group dimensions of multivariate params on output plots

* (R/cmd) have output digits depend on se?

* (R/cmd) check initial vals and gradients for finiteness and
  return error message if not


Web Pages
======================================================================

Manuals
======================================================================

* Explain how to do BUGS-like cut ops in Stan
  data {
    int<lower=0> N;
    real y[N];
    real x[N];
    int<lower=0> N_cut;
    real x_cut[N_cut];
  }
  parameters {
     real alpha;
     real beta;
     real<lower=0> sigma;
  }
  model {
    for (n in 1:N)
      y[n] ~ normal(alpha + beta * x[n], sigma);
  }
  generated quantities {
    real y_cut[N_cut];
    for (m in 1:M)
      y_cut[m] <- alpha + beta * x_cut[m] * beta;
  }

  With our next release, you can replace the last line with
  this:

    y_cut[m] <- normal_rng(alpha + beta * x_cut[m], sigma);

* Talk about index order and read order for matrix[I,J] a[K];
  -- in section on arrays in language reference
  -- in section on I/O
  -- example:
     real x[I,J,K];
     matrix[J,K] m[I];

    for (k in 1:K)
      for (j in 1:J)
        for (i in 1:I)
          x[i,j,k] = next...

* Stan's default is to use uniform inits between [-2,2] 
  on the unconstrained params.  For bounded params, that amounts
  to a range of (inv_logit(-2), inv_logit(2)) = (.12,.88), which
  is the central 80% interval.

  For positive params, that amounts to sampling between
  (exp(-2),exp(2)).

  For unconstrained params, sampling is between (-2,2).

  Plot these out because they're non-linear.

* transforms a la Matt, at least for all the normal-based cases
  we know of

* chapter on change of variables in the user guide

* more explanation of overall output analysis
  -- more on using generated quantities for posterior
     + predictive simulation (waiting for RNGs)
     + event probability calculations
  -- exp(lp__) propto posterior
  ?? do we do this in R or what?

* mention that discrete output functions like floor()
  lose gradient information

* copy FAQ into manual appendix (and onto web?)

* mention that even if you aren't interested in all the parameters,
  you do want to save them to monitor convergence and ESS

* provide mathematical definitions of all the special functions

* document version numbering: major.minor.patch
  - patch: just bug fixes, no behavior change
  - minor: some new functions, backward compatible
  - major: big changes, breaking backward compatibility

* add prior to getting started examples in doc

* use "draw" for a single parameter value and "sample" for
  multiples, with "sample of size 1000" or "1000 draws".

* drop Lucida console font section (?) or link Bigelow and Holmes

* break out sampler chapter in doc to describe multiple samplers

* performance tips
  - compute once and re-use if possible
  - pull out constants from loops
  - pull out terms from looped probabilities

* add example calculating log likelihood for DIC
  - how to specify likelihood terms in model?

* add example on individual log likelihoods for WAIC
  - how to specify individual likelihood terms in model?

   generated quantities {
    for (i in 1:I) 
      for (j in 1:J) 
        ll_y[i,j] <- normal_log(y[i,j]...)


* Doc the generated C++ model class

* Add an Index

* Description of reverse-mode auto-dif or too C++-like?








Release Management
======================================================================

* Jeffrey Arnold (8/13/12 message to list) about syntax highlighting.
  ? how to integrate

* Branch versions to make it easier to divide-and-conquer debugs
  and to allow developers to work from latest stable version

* appoint a commit manager


Models
======================================================================

* convergence diagnostics
    verboseEvolve:
      Evolve the current state by one half momentum step, one full
      position step, and then one half momentum step, displaying H, and
      ( \Delta H / epsilon^{2} ) at each update.  The idea is that if
      the leapfrog is working properly then you should see the
      cancellation of errors (some error in one direction, then lots of
      error in the other direction, then error in the first direction
      again to cancel the bulk of the difference).
  checkIntError: 
      Evolve the current state by a given number of iterations,
      displaying H and ( \Delta H / epsilon^{2} ) at each update so
      that you can see if the error is only slowly increasing or
      straight up diverging for the given evolution parameters.
  saveTrajectory:
      Evolve the current state through a single trajectory, saving each
      update to a file.  This way you can plot the entire trajectory and
      check it visually for problems (helpful in diagnosing divergences,
      bad step-sizes, etc).
  plotSamples:
      Save the proposed sample, proposed potential, and acceptance
      probability in the chain, then display the visual diagnostics
      we've discussed before, namely histograms of each marginal
      distribution for samples with p(accept) < 0.1 and p(accept) > 0.9.
      For NUTS you'd have to play around with this a bit -- Matt had
      recommended looking at the average acceptance probability across
      the entire chain, but I'm not sure which sample you'd plot in that
      case.  You'd do just as well plotting each step along the
      trajectory along with what its acceptance probability would be,
      provided the storage of all those samples doesn't cause issues.

* unit K-hyperball complement for stationary AR(K) parameters
  - what should the prior be?

* user model library
  - something we don't have to get too involved vetting
  - ideally things hard to do with other systems
  - or illustrate the unusual parts of our modeling language

* Online books with models based on BUGS:
  http://stat-athens.aueb.gr/~jbn/winbugs_book/
  http://137.227.242.23/pubanalysis/kerybook/

* deal with autocorrelation model params where require
  all roots of 1 - SUM_i rho[i]**i to be within unit
  complex circle

* show how to code undirected graphical models in Stan

* BUGS Models
  All of the vol1 -- vol3 models are implemented except
  for the following, categorized by issue.

  Sampler too slow
  - vol2/schools/schools.stan.0
  - vol3/fire/fire.stan.0

  Sampler hangs
  - vol3/funshapes/hsquare.stan.0
  - vol3/funshapes/ring.stan.0
  - vol3/funshapes/squaremc.stan.0

  Zero Poisson, 0 ~ Poisson(0 * parameters) hangs the following:
  Isn't this one fixed?
  - vol1/leuk/leuk.stan.0
  - vol1/leukfr/leukfr.stan.0

  Cyclic DAGs
  Can probably do these in Stan (not supported in JAGS).
  - vol2/ice/pineapple_ice.stan.0
  - vol2/ice/vanilla_ice.stan.0
  - vol3/jama/jama.stan.0

  Discrete parameters
  - vol2/asia/asia.stan.0
  - vol2/biopsies/biopsies.stan.0
  - vol2/stagnant/stagnant.stan.0
  - vol2/t_df/estdof2.stan.0

  Binary parameters
  Other versions of model working integrating out discretes.
  - vol2/cervix/cervix.stan.0 
  - vol2/hearts/hearts.stan.0

  Works, but data structure difficult to deal with
  - vol1/bones/bones.stan.0
```