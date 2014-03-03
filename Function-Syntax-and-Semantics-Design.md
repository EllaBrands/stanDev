### Function Block

* A block for function definitions will be optional before the data declaration block:

    ```
    functions {
       ... function definitions ...
    }
    data {
    ...
    ```

* Any number of functions (including zero) may be defined in the function-definition block.

* Mutual recursion will be allowable with (forward) declarations
    * have to wait until end of the function-definition block in order to guarantee all functions will be defined

* Given a function name and argument type sequence (i.e., the function signature), the return type must be unique
    * validate this uniqueness through the symbol table of functions, which must be updated as functions are declared


### Syntax of Function Declarations and Definitions

* Functions may be declared with a C-like declaration syntax for their signatures, e.g.,

    ```
    real foo(real, real);
    ````

* Functions will be defined with a C-like function syntax, e.g.,

    ```    
    real foo(real x, real y) {
      return x * y + 2;
    }
    ```

* If a function has not been declared, its definition will also constitute its declaration.

* Like other Stan functions, arguments and return types will not have syntactic type constraints;  if there are type constraints, they should be validated as part of the function.
    * In order to validate, we will need to add a raise-exception statement, which in C++ will raise a Stan-language-specific exception and will be caught, reported, and cause a sample rejection just like other math exceptions
    * The syntax can be something like
        ```
        exception("N must be positive, but found N=",N);
        ```
following the syntax of the print statement, which means it can take any number of arguments, the values of which will be concatenated together inot a string.

### Container Sizing in Argument and Return Types

* Containers will be declared with their sizes, as in Stan's modeling language, e.g.,

    ```
    vector[K] foo(int K, vector[K] v);
    ```
* All constants used for sizing must be declared as arguments before the types in which they appear;  so it would not be legal to reverse the order of `K` and `v` above.

* The return type logically follows the argument types in order for its size to be specified.

* ALTERNATIVE:  An option that differs from other Stan variable declarations would be to allow containers such as vectors and arrays to be declared without sizes, e.g.,
    ```
    vector[] bar(vector[] a[], matrix[,] b[,]);
    ```
An even more minimal approach would also remove the brackets, e.g.,
    ```
    vector bar(vector a[], matrix b[,]);
    ```
This last approach is cleanest, but most unlike Stan's existing variable declaration syntax.
    * A related alternative would be to allow Stan variables to be declared without sizes in the data block, so that they could be read in that way.  Users have asked for this before.

### Variable Scope and Local Variables

* Local variables may be declared in a function definition's body as in any other block, e.g.,

    ```
    real foo(real a, int K) {
      vector[K] v;
      matrix[K,K] m;
      ...
    }
    ```
Local block declarations work as usual in Stan, requiring sizes to be specified with variables that exist in scope. 

* The log probability accumulator, `lp__`, is not in scope for functions (see below for submodels)

* The log probability increment statement, `increment_log_prob(lp)`, is not allowed in functions (though see the section on submodels below).

* The PRNG seed will only be available in functions declared with names ending in `-rng`, like in our existing functions
    * in such functions, the existing PRNG functions will be callable
    * PRNG functions won't be callable elsewhere
    * submodels will not allow calling of PRNGs

### Return Statements

* Return statements are allowed anywhere in functions

* In order to pass the C++ compiler without warnings and provide warnings ourselves, we should try to pick up programs that have executions ending in something other than a return.  This can only be heuristic due to undecidability.  

* Syntactically, we will check that the final statement in a program (including all of its conditional branches) end in a return statement.  This is stricter than C++, which issues a warning and then has unspecified behavior. Ending in a return statement is defined recursively by
    * a return statement ends in a return statement
    * an exception ends in a return statement
    * a block ends in a return statement if its final statement ends in a return statement
    * a conditional ends in a return statement if each of its conditional blocks ends in a return statement and there is a default block that also ends in a return statement
    * a while loop ends in a return statement if its body block ends in a return statement
    * a function call ends in a return statement

* ALTERNATIVE: (suggested by Daniel) Just add a final statement that throws an exception, which can be done syntactically without warnings by adding
    ```
    if (true) throw_no_return(function_name);
    ```
where the support function is defined by

   ```
    void throw_no_return(const std::string& function_name) {
      std::string msg("reached end of function '");
      msg += function_name;
      msg += "' without returning";
      throw std::logic_error(msg);
    } 
    ```

### Input and Output Validation

* The proposed implementation will allow constrained input or output types
    * constrained input types allow any conforming expression, not just variables
    * inputs will be checked on function calls, raising `std::illegal_argument` exception if violated
    * ouptuts will be checked on return, also raising `std::illegal_argument` exception if violated

* This requires the program itself to validate it's output.

* ALTERNATIVE:  If we allow declarations of arguments and return values as simplex, cov_matrix, etc., then
    * inputs should be validated on input
    * outputs should be validated on return
    * if an illegal input or output is encountered, an exception should be raised

### Function application is an expression

* User-defined functions act just like other functions in that their application to arguments is an expression syntactically.

* Argument types will be validated syntactically by the parser for user-defined functions just like any other function

* Functions will not be usable as expressions;  see the section on submodels below

### Submodels

* Stan will allow submodels that will be declared like functions but with no return type

* Submodels will be declared in their own block after the functions block and before the data block, e.g.,

    ```
    submodels {
      normal(...) { ... }
    }

* Submodels will allow return statements without a value. 

    ```
    return;
    ```


* Submodels do not require a return;  a final return is implicit

* The log probability increment statement, `increment_log_prob()`, is legal in submodels but not in functions.  
    * submodels with that increment the log prob accumulator need to get expanded at compile time to take the `lp__` variable as their first or last argument and then to be passed that value automatically when called.
    * submodels that involve log probabilities will not be allowed in blocks where `lp__` is not defined.

* Here's an example of how a submodel would be declared (the scale should get a lower bound if constraints are allowed)

    ```
    linear_regression(vector y, vector x, real alpha, real beta, real sigma, vector x) {
      sigma ~ cauchy(0, 2.5);
      alpha ~ cauchy(0, 2.5);
      beta ~ cauchy(0,2.5);
      y ~ normal(alpha + beta * x, sigma);
    }
    ```

### Call by Constant Reference

* Functions and submodels will be called by constant reference, i.e., declared in C++ as `const T&` for whatever type `T` is used for the arguments

* ALTERNATIVE:  Marcus suggested calling by non-constant reference in order to more easily support multiple returns without additional data types.  I'm afraid that this will confuse users in the declarations, but maybe not.  If we start with only constant references, then we'd have to mark the non-constant ones later, which is the reverse of the C/C++ convention.

### FUTURE:  Defining New Probability Functions

* We could follow existing practice of taking any function ending in `_log` to define a new probability density.  So if we want to define y in terms of regression coefficient and noise scale, it'd be
    ```
    void linear_regression_log(real y, vector x, real alpha, real beta, real sigma) {
      y ~ normal(alpha + x * beta, sigma);
    }
    ```
and then it would get called as

    ```
    y ~ linear_regression(x, alpha, beta, sigma);
    ```

* If we define a new probability function, it should also be available as a function returning its log prob
    * Most natural is to have it define a log prob that just gets incremented in the sampling statement call and returned in the usual call --- can just follow existing pattern here


### FUTURE: Multiple Return Values

* It's both awkward and inefficient that we now have two functions for eigendecompisitions, eigenvectors and eigenvalues.  It'd be nice to have a way to do something like Python does and return lists.

    ```
    (vector,matrix) eigendecompose(matrix x) {
      vector eigenval;
      matrix eigenvecs;
      eigenvals <- eigenvalues(x);
      eigenvecs <- eigenvectors(x);
      return (eigenvals,eigenvecs);
    }
    ```

* The C++ type can be implemented with the <a href="http://www.boost.org/doc/libs/1_55_0/libs/tuple/doc/tuple_users_guide.html">Boost Tuple Library</a>.

* If we have a general list type, then we can use a literal syntax like `(x,y,z)` to create a list.
    * While we're at it, a syntax to create vectors, such as `[x y z]`, would also be nice;  it would create a column vector and a row vector would be called out as `[x y z]'`.
    * And while we're at that, might as well do matrices, too, as `[[a b c] [d e f]]` for a 2 x 3 matrix.

* We'd want to define a syntax for declaring lists, say

    ```
    (vector[K], matrix[K,K]) eigendecomposition;
    ```
and then allow access to the vector as `eigendecomposition[1]` and `eigendecomposition[2]`.

* If we want to define joint densities, we can then just use the built-in type declaration syntax and literal syntax, e.g.,

    ```
    void linear_regression_log((vector, real, real,real) z, vector x) {
      vector y;
      real alpha;
      real beta;
      real sigma;           
      y <- z[1];
      alpha <- z[2];
      beta <- z[3];
      sigma <- z[4]
      y ~ normal(alpha + beta * x, sigma);
      alpha ~ cauchy(0, 2.5);
      beta ~ cauchy(0,2.5);
      sigma ~ cauchy(0, 2.5);
    }
    ```
and

    ```
    (y,alpha,beta,sigma) ~ linear_regression(x);
    ```

#### FUTURE:  Solvers for Functions

* Along the lines of diffeq solvers, add a solver for functions, e.g. (from Ben) with a function
    ```
    real loss(real theta, real x, real y, real z);
    ```
allow a call like 

    ```
    theta <- minimize(theta, loss, x, y, z);
    ```

#### FUTURE: Unit testing framework

* (from Daniel) We want to have a unit-testing framework that can access functions in a standalone way
    * this should be possible because they will all be declared in the model namespace as staticsa

####  QUESTIONS

* (from Ben) Can we allow PRNGs so that we can implement a D.I.Y. Gibbs sampler?  (Probably not.)


