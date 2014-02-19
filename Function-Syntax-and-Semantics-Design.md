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
If a function has not been declared, the definition will also constitute its declaration.

* Like other Stan functions, arguments and return types will not have syntactic type constraints;  if there are type constraints, they should be validated as part of the function.
    * In order to validate, we will need to add a raise-exception statement, which in C++ will raise a Stan-language-specific exception and will be caught, reported, and cause a sample rejection just like other math exceptions
    * The syntax can be something like
        ```
        exception("N must be positive, but found N=",N);
        ```
following the print statement, which means it can take any number of arguments, the values of which will be concatenated together

### Container Sizing in Argument and Return Types

* Containers will be declared with their sizes, as in Stan's modeling language, e.g.,

    ```
    vector[K] foo(int K, vector[K] v);
    ```
* All constants used for sizing must be declared as arguments before the types in which they appear;  so it would not be legal to reverse the order of `K` and `v` above.

* The return type logically follows the argument types in order for its size to be specified.

* An alternative that will be harder to implement and not provide a such error checking would be to allow containers such as vectors and arrays to be declared without sizes, e.g.,
    ```
    vector[] bar(vector[] a[], matrix[,] b[,]);
    ```
An even more minimial approach would also remove the brackets, e.g.,
    ```
    vector bar(vector a[], matrix b[,]);
    ```
This last approach is cleanest, but most unlike Stan and most difficult to implement.

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

* The log probability accumulator, `lp__`, is not in scope for functions (see below for subroutines)

* The log probability increment statement, `increment_log_prob(lp)`, is not allowed in functions (though see the section on subroutines below).

### Return Statements

* Return statements are allowed anywhere in functions

* In order to satisfy the C++ compiler, we will need to test that every execution branch has a return statement;  Need to test that there is a valid return from every branch of execution.  This is more subtle than just requiring one at the end, because of cases like

    ```
    real bar(real x) {
      if (x < 1e-28)
        return x;
      else
        return x * x;
    }
    ```
where both branches of a conditional have a return.  If there is a final statement outside of a conditional, it must be a return.  

* It's even trickier for a loop

    ```
    for (n in 1:N)
      if (n > 100) return x;
    ```
which isn't guaranteed to have a return unless N > 100.  While statements present similar issues.  

* We could simplify by just returning a default value of the return type if the program exits without a return.  This could be coded in C++ after a return with something like:

    ```
    if (true) return default_simplex();
    ```
where it's the minimal size meeting all the constraints (treat like zero-inits).


### Input and Output Validation

* If we allow declarations of arguments and return values as simplex, cov_matrix, etc., then
    * inputs should be validated on input
    * outputs should be validated on return

* If an illegal input or output is encountered, an exception should be raised

### Void Type for Returns

* There is no reason to include a void return type for functions, because they have no side effects.  

### Function application is an expression

* User-defined functions act just like other functions in that their application to arguments is an expression.

* Argument types need to be validated syntactically

### Subroutines

* Subroutines act as statements, not as expressionss;  functions act 

* Should be declared with a void type, or just no return type at all

* Should allow return statements with no values, e.g.,

    ```
    return;
    ```
and optionally a void return 
    ```
    return void;

* The log probability increment statement, `increment_log_prob()`, should be legal in subroutines but not in functions.  Functions with an increment log prob need to get expanded at compile time to take the `lp__` variable as their first or last argument and then to be passed that value automatically when called.

* Here's an example of how a subroutine would be declared (the scale should get a lower bound if constraints are allowed)

    ```
    void linear_regression(vector y, vector x, real alpha, real beta, real sigma, vector x) {
      sigma ~ cauchy(0, 2.5);
      alpha ~ cauchy(0, 2.5);
      beta ~ cauchy(0,2.5);
      y ~ normal(alpha + beta * x, sigma);
    }
    ```


### Defining New Probability Functions

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


### Multiple Return Values

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

* The C++ type can be a structure with numbered fields, as in

    ```
    (real, real, real[])
    ```
picking out

    ```
    template <typename T1, typename T2, typename T3>
    struct list_type_1 {
      T1 one;
      T2 two;
      std::vector<T3> three;
    };
    ```
and then translating ```a[2]``` in Stan syntax as ```a.two```, etc.

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


### Call by Constant Reference

* Functions and statements will be called by constant reference, i.e., declared in C++ as `const T&` for whatever type `T` is used
