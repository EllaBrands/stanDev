# Intro

The ability to do ODEs has made Stan very attractive to many researchers. But there are a lot more things that could be done with user-defined functions (see below for examples); functions that apply to other functions are often called "functionals" and we will use that terminology here to separate them from first order functions that operate on integer and real values. 

## Issues

The following issues apply ode_integrate and will also apply to new functionals:

0. Stan does not support types for functions or functions as first-class objects, so there is no way to declare the type of a function or functional other than implicitly through its signature.

0. Without function typing, coding new functionals is awkward because it has to be done directly in the C++ grammars for statements because unlike regular functions, there's no typing for functionals built into Stan.

0. Passing arguments is awkward when using `integrate_ode` because all data have to be packed into a `real[]` and / or an `int[]`;  this stems from two issues
    
    0. Functions can't access other Stan variables (like data) because they are defined first and have lexical scope.

    0. Stan doesn't support polymorphism in functionals so the function argument to `ode_integrate` has a fixed type.

0.  Like with ODEs, there may be a lot of foreseeable and unforeseen issues with numerical stability, execution speed, posterior geometry, etc.

# Scope

At the moment, user-defined functions have [lexical/static scope](https://en.wikipedia.org/wiki/Scope_(computer_science)#Lexical_scoping) (as in Java/C/C++) and also appear before any other variable declarations.  This has two consequences:

* CON: all of their arguments must be passed to them explicitly

* PRO: they are completely encapsulated by their definitions


While this is tolerable for a lot of ODE problems, it would be tedious if the problem required matrices of different sizes. It gets worse if we contemplate supporting other functionals that do not have as much structure as ODEs.

One solution to this problem would be to make user-defined function (capable of being) class functions or friend functions of the class so that they could access things declared in the `data` block, the `transformed data` block, and the `parameters` block without having to explicitly pass them to the user-defined function.

# Possible Functionals

Here are some additional functionals that we may want to expose in the Stan language.

## Gradients

In some dynamic problems, the likelihood of the data depends on derivatives.

```
real foo(vector theta);
...
vector[K] g;
g <- gradient(foo, theta);
```

## Jacobians

In some dynamic problems, the likelihood of the data depends on derivatives. Also, if a user does a transformation that requires adjusting `lp__` by the logarithm of the absolute value of the determinant of the Jacobian matrix of the inverse transformation, it would be useful to be able to produce that Jacobian matrix for cases where the determinant does not take a simple analytical form.
```
vector foo(vector theta);
...
matrix[K,K] J;
J <- jacobian(foo, theta);
```

## Optimization

Many users have asked for the ability to implicitly define transformed parameters as the solution to an optimization problem. This will require some implicit differentiation internally in order to correctly calculate the derivatives of `lp__` with respect to the parameters that go through `theta_star`. In addition, we may have to do more to make Stan's optimization algorithms stand-alone. 
```
real foo(vector theta);
...
vector[K] theta_star;
theta_star <- rep_vector(0, K);          // starting values
theta_star <- minimize(foo, theta_star); // or maximize(foo, theta_star)
```

## Roots

Many users have asked for the ability to implicitly define transformed parameters as the solution to a system of equations. This entails the same considerations as optimization. Also, the solution may only be unique within an interval, so we need to do transformations from the real line to a subset of the real line.
```
real foo(vector theta);
...
vector[K] theta_root;
theta_root <- rep_vector(0, K); // starting values
theta_root <- find_root(foo, theta_root, lower, upper);
```

## Covariance functions

We get asked about this a lot.
```
matrix foo(vector theta);
y ~ gp(zeros, foo, theta);
```

## Arbitrary Integrals

Marco has been working on this. Here a key question is getting the error bound on the integral to be small enough for the outside algorithm (MCMC, etc.) to work well. Also, some of the derivatives may be brutal in which case the user may want to pass in another function to calculate the gradient.
```
real foo(real theta);
...
real area;
area <- integrate(foo, lower, upper, tolerance);
```
Ultimately, we will probably need different numerical integration schemes, such as `integrate_gauss_hermite`, etc.

## Specialized Integrals

There are also some integrals where we should be using a specialized numerical algorithm to do the integration.

### FFTs

### Characteristic functions
