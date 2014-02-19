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


### Function Declarations and Definition Syntax

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

* There are three ways that make sense to define container arguments, but only one of them should be allowable.
    1. Containers are declared with sizes as in Stan, e.g.,

        ```
        vector[K] foo(int<lower=0> K, vector[K] v);
        ```
Note that the return type is logically after the argument types in order to allow sizing to be specified.  Otherwise, size variables must appear before the elements which they size, just as in the rest of Stan.
    2. Sizes of array, vector, and matrix types are not included, but all of the brackets that would otherwise be used in other Stan blocks will be required, e.g., 

        ```
        vector[] bar(vector[] a[], matrix[,] b[,]);
        ```
    3. The most minimal approach is like the previous approach, but without the brackets on the tpes themselves, e.g., 

        ```
        vector bar(vector a[], matrix b[,]);
        ```
This is somehow the simplest, but the least like the rest of Stan and therefore a bit more complicated to document.

### Variable Scope and Local Variables

* Local variables may be declared in a function definition's body as in any other block, e.g.,

    ```
    real foo(real a, int K) {
      vector[K] v;
      matrix[K,K] m;
      ...
    }
    ```
Local block declaration is as for all other Stan variables, so that it requires a size.

### Return Statements



### Void Type for Returns

* There is no reason to include a void return type for functions, because they have no side effects.  

### Log Probability Accessible?

I think we need to make an exception to the rule of "no global variables" for the implicit log probability accumulator.  I think it makes sense to allow functions to take that in as an argument.  

```
void linear_regression(vector y, vector x, real alpha, real beta, real<lower=0> sigma) {
  sigma ~ cauchy(0, 2.5);
  alpha ~ cauchy(0, 2.5);
  beta ~ cauchy(0,2.5);
  for (n in 1:size(x))
    y[n] ~ normal(alpha + beta * x, sigma);
}
```

Or maybe there would be a different syntax for defining new distributions?  I'm not sure how to call them out.  Maybe new blocks?  I'm thinking something along the lines of

```
probability functions {
  linear_regression(vector y | vector x, real alpha, real beta, real<lower=0> sigma) {
    for (n in 1:size(x))
      y[n] ~ normal(alpha + beta * x, sigma);   
  }
}
```

Then it would be called in code using something like

```
y ~ linear_regression(x,alpha,beta,sigma);
```
  
Note the use of a vertical bar and no prior for the parameters `alpha`, `beta`, and `sigma`.  If we want to have it be a joint probability function, we'd need something like

```
probability functions {
  linear_regression(vector y, real alpha, real beta, real<lower=0> sigma | vector x) {
    sigma ~ cauchy(0, 2.5);
    alpha ~ cauchy(0, 2.5);
    beta ~ cauchy(0,2.5);
    for (n in 1:size(x))
      y[n] ~ normal(alpha + beta * x, sigma);
}
```

and then it would need to be called using a list-like syntax

```
(y,alpha,beta,sigma) ~ linear_regression(x);
```

In order to do any of this, we need to have the log probability function accessible.

### Local Variables

Is it OK to have them just at the top of blocks as we have now?  It was the lazy thing to do when I implemented blocks the first time, but I sort of regret it because I like to declare variables near where I use them.  I suppose this issue's really independent if we just treat a function body like any other block.

### Array Sizing

Two possibilities seem obvious, the first of which is to
declare with no sizes, as in:

```
real foo(vector x) {
  real sum;
  for (i in 1:size(x))
    sum <- sum + x[i];
    return sum;
  }
}
```

The other would be to require sizes, so we'd have:

```
real foo(int<lower=0> N, vector[N] x) {
  real sum;
  for (i in 1:N)
    sum <- sum + x[i];
  return sum;
}
```

The latter seems like a real pain and wouldn't give us the generality to define all the built-in functions within Stan (which seems like a drawback in principle).  The upside is that it automatically gives us size error checking.  

And for the second option, it'd be easier to require sizes to appear in the argument list before the vectors they size.

###  Multiple Return Values

It's both awkward and inefficient that we now have two functions for eigendecompisitions, eigenvectors and eigenvalues.  It'd be nice to have a way to do something like Python does and return lists.

```
list[vector,matrix] eigendecompose(matrix x) {
  vector eigenval;
   matrix eigenvecs;
   eigenvals <- eigenvalues(x);
   eigenvecs <- eigenvectors(x);
   return (eigenvals,eigenvecs);
}
```

This is another issue that's probably independent.  

If we have a general list type, then we can use `(x,y,z)` syntax to create a list.  (As an aside, it'd be nice to have a syntax like this to create vectors, maybe something like `[x y z]`.)

Another option would be to require lists to be declared and then use setters.

```
list[ vector[K], matrix[K,K] ] eigendecomp;
eigendecomp[1] <- eigenvals;
eigendecomp[2] <- eigenvecs;
```

### Call-by-Reference vs. Call-by-Const-Reference

I was thinking functions would translate into C++ as call by constant reference.  

An alternative that would allow multiple returns would be to drop the implicit const-ness, and allow

```
void eigendecompose(matrix x, vector eigenvals, matrix eigenvecs) {
  eigenvals <- eigenvalues(x);
  eigenvecs <- eigenvectors(x);
}
```

We could also use a `const` syntax, but then that's starting to get complicated for users.




### Global Variables?

Let's just say "no" to global variables being used in function definitions.  It's how R works and how BUGS defines functions, but I don't think it's a good idea.