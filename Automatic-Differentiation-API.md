Stan's reverse-mode automatic differentiation library is designed to be usable, extensive and extensible, efficient, scalable, stable, portable, and redistributable. This page describes the installation and usage of the API.

Forward-mode automatic differentiation is under development.

# Outline

1. [Installation](#installation)
2. [Basic usage](#basic-usage)
3. [The Stan Math Library](#the-stan-math-library)
4. [Advanced usage](#advanced-usage)
5. [Troubleshooting](#troubleshooting)


# Installation

Stan's autodiff library is header-only, which makes it trivial to install.

## 1. Download Stan

Download and unpack Stan in a place where you can access it. We'll refer to the unpacked directory as `STAN_HOME`.

Download one of the packages below or clone it directly from [GitHub](https://github.com/stan-dev/stan).

|  Latest Tagged Version <br /> <small>(suggested)<small /> |  Development Version <br /> <small>(experimental)<small />|
|------------------------|-----------------------|
| **won't work with these instructions yet** [stan-2.6.3.zip](https://github.com/stan-dev/stan/archive/v2.6.3.zip) | [stan-develop.zip](https://github.com/stan-dev/stan/archive/develop.zip) |
| **won't work with these instructions yet** [stan-2.6.3.tar.gz](https://github.com/stan-dev/stan/archive/v2.6.3.tar.gz) |  [stan-develop.tar.gz](https://github.com/stan-dev/stan/archive/develop.tar.gz) |


## 2. Building with Stan

To build with Stan's autodiff, we'll need to tell the compiler where to find the autodiff header files. Stan's autodiff has two dependencies, Boost and Eigen, which are distributed with the download if you don't already include these libraries.

Below are two sets of instructions, one for calling the compiler directly and the other for using `make`.

### Calling `g++` or `clang++` directly 

Add this to your compiler line, where `STAN_HOME` is replaced with the appropriate directory:

```
-I STAN_HOME/src -isystem STAN_HOME/lib/boost_1.55.0 -isystem STAN_HOME/lib/eigen_3.2.4
```

### Calling a compiler through `make`

Add these lines to your makefile, where `STAN_HOME` is replaced with the appropriate directory:

```
CXXFLAGS += -I STAN_HOME/src 
CXXFLAGS += -isystem STAN_HOME/lib/boost_1.55.0 
CXXFLAGS += -isystem STAN_HOME/lib/eigen_3.2.4
```

Note: this will only work if you're using the standard makefile conventions to build your executable. It doesn't matter where in the makefile this goes -- makefiles aren't imperative.


### Explanation

All we need to do to use Stan is tell the C++ compiler where to find the Stan header files and the library files.

* Include the Stan directory: `-I STAN_HOME/src`
* Include the Boost library: `-isystem STAN_HOME/lib/boost_1.55.0`
* Include the Eigen library: `-isystem STAN_HOME/lib/eigen_3.2.4`

If you already have Boost or Eigen included, exclude the appropriate parts.


## 3. Including Stan source

To include the Stan reverse-mode autodiff library into your code, add these includes to your source code, in this order:

```
#include <stan/math/prim/arr/meta.hpp>
#include <stan/math/prim/mat/meta.hpp>
#include <stan/math/prim/scal/meta.hpp>
#include <stan/math/rev/scal/meta.hpp>
#include <stan/math/prim.hpp>
#include <stan/math/rev.hpp>
```

**Note**: this will be simpler in the future; we will be providing a single include that will take care of including all these files in this order.

## 4. Verify Installation

Place this minimal Stan example in `stan_example.cpp`


[//]: # (stan_example1.cpp)
```
#include <stan/math/prim/arr/meta.hpp>
#include <stan/math/prim/mat/meta.hpp>
#include <stan/math/prim/scal/meta.hpp>
#include <stan/math/rev/scal/meta.hpp>
#include <stan/math/prim.hpp>
#include <stan/math/rev.hpp>
#include <iostream>

int main() {
  std::cout << "Hello Stan!" << std::endl;
  return 0;
}
```

Verify that you can compile this. If you're using `make`, you can compile this with a one-liner:

```
> make example CXXFLAGS="-I STAN_HOME/src -isystem STAN_HOME/lib/boost_1.55.0 -isystem STAN_HOME/lib/eigen_3.2.4"
```

If you can compile this, you're successful. If you get compiler errors, please check paths to the libraries.

## 5. Checklist

- [ ] Download and unpacked source. We're calling this location `STAN_HOME`.
- [ ] Add include options for the compiler to find the Stan autodiff library.
- [ ] Compile the simple model.


# Basic Usage

We'll start by describing how to use the Stan reverse-mode autodiff library in simple examples. In the next section, [Advanced Usage](#advanced-usage), we'll walk through some of the finer details of Stan's reverse-mode autodiff API.

## Before You Begin

By this point, you should have followed all the [installation instructions](#installation). We will continue to extend `stan_example.cpp` from the installation isntructions.


## `stan::agrad::var`

Stan's autodiff type is `stan::agrad::var`. Use `stan::agrad::var` for any of the independent variables (i.e. the variables you want to take derivatives with respect to) and the dependent variable (i.e. the final expression).

Stan's autodiff is implemented by overloading operators. Practically, this means that you can use a `stan::agrad::var` in place of `double` for the operators (e.g. `+`, `-`, `*`, `/`, etc.). There is automatic promotion to a `stan::agrad::var` from a primitive, but there is no automatic demotion.

We'll start with simple examples that build mathematical expressions using `stan::agrad::var`. We'll show how to take the gradient in the next subsection. For these next examples, replace the `main` function with these blocks.


### Simple Expressions

We'll start with a simple expression with primitives:

```
int main() {
  double x, y, z, a;

  x = 2.5;
  y = 4.2;
  z = x * y + y * y * y;
  z -= 5.0;

  std::cout << "  x: " << x << std::endl
            << "  y: " << y << std::endl
            << "  z: " << z << std::endl
            << "  a: " << a << std::endl;
  return 0;
}

```

The output is fairly straightforward, but `a` isn't guaranteed to be initialized to anything sensible:

```
  x: 2.5
  y: 4.2
  z: 79.588
  a: 6.95321e-310
```


What would this look like with Stan autodiff types? All we have to do is replace `double` with `stan::agrad::var`:


[//]: # (stan_example2.cpp)
```
int main() {
  stan::agrad::var x, y, z, a;

  x = 2.5;
  y = 4.2;
  z = x * y + y * y * y;
  z -= 5.0;

  std::cout << "  x: " << x << std::endl
            << "  y: " << y << std::endl
            << "  z: " << z << std::endl
            << "  a: " << a << std::endl;
  return 0;
}
```

The resulting output is the same, except for `a`, which hasn't been initialized:

```
  x: 2.5
  y: 4.2
  z: 79.588
  a: uninitialized
```


### Demotion: Compiler Error!

This will probably be the first compiler error you'll see. You can't assign a `stan::agrad::var` to a `double`! Any expressions containing a `stan::agrad::var` will be promoted to an autodiff type, so when you come across this error, it may be embedded in a compound expression.

[//]: # (stan_example3.cpp)
```
int main() {
  stan::agrad::var x;
  double y;

  x = 2.5;
  y = 1.0 + x;

  return 0;
}
```

Here's the compiler error I get from `clang++`:

```
stan_example.cpp:14:5: error: assigning to 'double' from incompatible type 'stan::agrad::var'
  y = 1.0 + x;
    ^ ~~~~~~~
```

There's nothing to fear; the message is telling us that we can't assign a `stan::agrad::var` to `double`. One fix we can make is to change the declaration of `y` to a `stan::agrad::var`.

We can also extract the value of `x` using the `val()` method. This returns a `double`.

```
int main() {
  stan::agrad::var x;
  double y;

  x = 2.5;
  y = 1.0 + x.val();

  return 0;
}
```



### Uninitialized `stan::agrad::var`: Segementation Fault!

Uninitialized autodiff variables used in expressions can cause segfaults. Stan's autodiff is implemented with pointers under the hood and rather than be safe and checking for validity of autodiff   variables at every turn, we expect the user of the API to have initialized each variable.

Here's a quick example that will compile:

[//]: # (stan_example4.cpp)
```
int main() {
  stan::agrad::var x, y;
  y = 1.0 + x;
  return 0;
}
```

Result:

```
> ./stan_example
Segmentation fault: 11
```

If you see this, first thing to do is check that every `stan::agrad::var` has been initialized. It's ok that uninitialized variables exist; they just can't be used for computation.


## Gradients!!!

Now that we know how to build basic expressions using Stan, let's use autodiff to get gradients! We'll start with the example at the start of this section:

```
x = 2.5;
y = 4.2;
z = x * y + y * y * y - 5.0;
```

`x` and `y` are the independent variables, `z` is the dependent variable. In this example, we are interested in these three things:

- the value of `z = 79.588`. We've already seen how we can access this using the `val()` method.
- the value of `dz / dx` evaluated at `x = 2.5` and `y = 4.2` `dz / dx = 4.2` (symbolic: `dz / dx = y`)
- the value of `dz / dy` evaluated at `x = 2.5` and `y = 4.2`. `dz / dy = 55.42` (symbolic: `dz / dy = x + 3 * y^2`)

There are multiple ways to get the gradients in Stan. We'll show the direct method here.

### Direct method for evaluating gradients

We need to do 3 things:

1. Create a `std::vector<stan::agrad::var>` that contains the independent variables.
2. Compute the gradients of the dependent variable with respect to the independent variables.
3. Recover the memory used. (Memory is managed by Stan; recovery doesn't actually "free" the memory, but allows it to be reused.)

This example, based on the first autodiff example above, shows how to do the three steps.

[//]: # (stan_example5.cpp)
```
int main() {
  stan::agrad::var x, y, z;

  x = 2.5;
  y = 4.2;
  z = x * y + y * y * y;
  z -= 5.0;

  // Step 1
  std::vector<stan::agrad::var> independent_vars;
  independent_vars.push_back(x);
  independent_vars.push_back(y);

  // container for the gradients
  std::vector<double> gradients;

  // Step 2
  z.grad(independent_vars, gradients);

  // Step 3
  stan::agrad::recover_memory();

  std::cout << "z:       " << z.val() << std::endl
            << "dz / dx: " << gradients[0] << std::endl
            << "dz / dy: " << gradients[1] << std::endl;
  return 0;
}
```

The output matches what we expect:

```
  z:       79.588
  dz / dx: 4.2
  dz / dy: 55.42
```

No matter how complicated the expression gets, this is how we can evaluate gradients. Make sure to call `stan::agrad::recover_memory()` after each gradient call, or you may find yourself running out of RAM.


# The Stan Math Library

The Stan math library is a set of functions functions that operates on `double` types and often are templated well enough to take on Stan's autodiff types. We'll discuss how to write code using both `double` and `stan::agrad::var` types, but in theory, other floating point types should be able to be substituted without a problem. 

We currently don't have a single list of functions in the math library, but most of the functions are documented in the Doxygen API documentation (html in the tagged Stan version) or the Stan Reference Manual.

A couple things to note:

* We try to be safe with our functions. The Stan math library will throw exceptions when the inputs to a function are not valid.
* A lot of our functions are vectorized. This means that you can often provide a vector or `Eigen::Matrix` that's shaped as either a vector or row-vector to our functions and it'll do the reasonable broadcasting internally.

## Namespaces

There are two namespaces for the Stan math library:

* `stan::math` contains templated functions that can be used for both `double` and, in most cases, for `stan::agrad::var` also.
* `stan::agrad` contains function overloads that can be used for `stan::agrad::var`. Included in this namespace are thing you would find in the `std` (standard) and global namespaces.

## General usage pattern

There will be many exceptions to this, but the general usage pattern will be to put a using statement for a particular function in `stan::math` and allow [argument dependent lookup](http://en.wikipedia.org/wiki/Argument-dependent_name_lookup) to find the right `stan::agrad` version for autodiff variables.

For example, if we wanted to use `stan::math::log1p`, we might do something like:

```
...
  using stan::math::log1p;
  
  y = log1p(x);
...

```
where `x` and `y` can be declared as `double` or `stan::agrad::var`. The next subsection will show examples of usage.


# Advanced Example

## Prerequisites

Before we start, you should be comfortable with:

1. Compiling a program with Stan's autodiff library.
2. Building a simple expression using Stan's autodiff types.
3. Calculating gradients and how to recover memory.

If you can't do any of the above, please go back to the previous sections.

## Example: log of the normal density function

Suppose we're interested in calculating the log of the [normal density](http://en.wikipedia.org/wiki/Normal_distribution#General_normal_distribution) function. Stan already has an efficient version of this implemented, but we'll use this as an example.

We'll evaluate this at two points:

* `y = 0`, `mu = 0`, `sigma = 1`, `foo(y, mu, sigma) = -0.918939`
* `y = 1`, `mu = 2`, `sigma = 3`, `foo(y, mu, sigma) = -2.07311`

### Example using `double`

We'll start with evaluating this function using `double`. We'll continue to extend the examples from above.

[//]: # (normal_example1.cpp)
```
...
#include <cmath>  // std::log

double foo(double y, double mu, double sigma) {
  using std::log;
  using stan::math::square;
  using stan::math::pi;
  
  return -0.5 * square((y - mu) / sigma) - log(sigma)
    - 0.5 * log(2 * pi());
}


int main() {
  double y1(0), mu1(0), sigma1(1);
  double lp1 = foo(y1, mu1, sigma1);

  std::cout << "foo(" << y1 << ", " << mu1
            << ", " << sigma1 << ") = " << lp1 << std::endl;

  double y2(1), mu2(2), sigma2(3);
  double lp2 = foo(y2, mu2, sigma2);

  std::cout << "foo(" << y2 << ", " << mu2
            << ", " << sigma2 << ") = " << lp2 << std::endl;
  return 0;
}
```

This example uses `std::log` brought in by the include to `cmath`, the `stan::math::square` function and the `stan::math::pi` function, which returns the constant.

### Example using `stan::agrad::var`

Changing everything to `stan::agrad::var` is simple. 

* Change all instances of `double` to `stan::agrad::var`. 
* We're now passing to the `foo()` function by reference (`stan::agrad::var&`).
* We've also added `const` to the arguments for const correctness.

Notice, the calls to functions in both the `std` and `stan::math` namespace have using statements. If you don't do this, you'll find that this example does not compile. `std::log` does not operate on `stan::agrad::var`, but argument dependent lookup will automatically use the `stan::agrad::log` function without having to specify it explicitly. 

[//]: # (normal_example2.cpp)
```
...
#include <cmath>  // std::log
#include <vector> // std::vector

stan::agrad::var foo(const stan::agrad::var& y,
                     const stan::agrad::var& mu,
                     const stan::agrad::var& sigma) {
  using std::log;
  using stan::math::square;
  using stan::math::pi;
  
  return -0.5 * square((y - mu) / sigma) - log(sigma)
    - 0.5 * log(2 * pi());
}

int main() {
  std::vector<stan::agrad::var> independent_vars;
  std::vector<double> gradients;

  stan::agrad::var y1(0), mu1(0), sigma1(1);
  stan::agrad::var lp1 = foo(y1, mu1, sigma1);
  independent_vars.push_back(y1);
  independent_vars.push_back(mu1);
  independent_vars.push_back(sigma1);

  lp1.grad(independent_vars, gradients);
  stan::agrad::recover_memory();
  std::cout << "foo(" << y1 << ", " << mu1
            << ", " << sigma1 << ") = " << lp1 << std::endl
            << "dfoo / dy1 = " << gradients[0] << std::endl
            << "dfoo / dmu1 = " << gradients[1] << std::endl
            << "dfoo / dsigma1 = " << gradients[2] << std::endl;

  stan::agrad::var y2(1), mu2(2), sigma2(3);
  stan::agrad::var lp2 = foo(y2, mu2, sigma2);
  independent_vars.clear();
  independent_vars.push_back(y2);
  independent_vars.push_back(mu2);
  independent_vars.push_back(sigma2);

  lp2.grad(independent_vars, gradients);
  stan::agrad::recover_memory();
  std::cout << "foo(" << y2 << ", " << mu2
            << ", " << sigma2 << ") = " << lp2 << std::endl
            << "dfoo / dy2 = " << gradients[0] << std::endl
            << "dfoo / dmu2 = " << gradients[1] << std::endl
            << "dfoo / dsigma2 = " << gradients[2] << std::endl;
  return 0;
}
```

Changing all uses of `double` to `stan::agrad::var` is easy. But this may not be the most efficient thing to do. Suppose we wanted to know the gradients with respect to `mu` and `sigma`, but not of `y`. In this case, we want to leave `y` as `double`, but the other two arguments as `stan::agrad::var`. The return type needs to be promoted to `stan::agrad::var` since at least one of the elements in the expression is `stan::agrad::var`.

We can add a function overload, leaving in the original function:

[//]: # (normal_example3.cpp)
```
...
#include <cmath>  // std::log
#include <vector> // std::vector

stan::agrad::var foo(const stan::agrad::var& y,
                     const stan::agrad::var& mu,
                     const stan::agrad::var& sigma) {
  using std::log;
  using stan::math::square;
  using stan::math::pi;
  
  return -0.5 * square((y - mu) / sigma) - log(sigma)
    - 0.5 * log(2 * pi());
}

stan::agrad::var foo(double y,
                     const stan::agrad::var& mu,
                     const stan::agrad::var& sigma) {
  using std::log;
  using stan::math::square;
  using stan::math::pi;
  
  return -0.5 * square((y - mu) / sigma) - log(sigma)
    - 0.5 * log(2 * pi());
}

int main() {
  std::vector<stan::agrad::var> independent_vars;
  std::vector<double> gradients;

  double y1(0);
  stan::agrad::var mu1(0), sigma1(1);
  stan::agrad::var lp1 = foo(y1, mu1, sigma1);
  independent_vars.push_back(mu1);
  independent_vars.push_back(sigma1);

  lp1.grad(independent_vars, gradients);
  stan::agrad::recover_memory();
  std::cout << "foo(" << y1 << ", " << mu1
            << ", " << sigma1 << ") = " << lp1 << std::endl
            << "dfoo / dmu1 = " << gradients[0] << std::endl
            << "dfoo / dsigma1 = " << gradients[1] << std::endl;

  stan::agrad::var y2(1), mu2(2), sigma2(3);
  stan::agrad::var lp2 = foo(y2, mu2, sigma2);
  independent_vars.clear();
  independent_vars.push_back(y2);
  independent_vars.push_back(mu2);
  independent_vars.push_back(sigma2);

  lp2.grad(independent_vars, gradients);
  stan::agrad::recover_memory();
  std::cout << "foo(" << y2 << ", " << mu2
            << ", " << sigma2 << ") = " << lp2 << std::endl
            << "dfoo / dy2 = " << gradients[0] << std::endl
            << "dfoo / dmu2 = " << gradients[1] << std::endl
            << "dfoo / dsigma2 = " << gradients[2] << std::endl;
  return 0;
}
```

In this example, we've the first call uses the function definition with `double` for the first argument and the second call uses the function definition with `stan::agrad::var&` for the first argument. In more complex examples, this may have a significant effect on computational time.

If you want to write function overloads that are maximally efficient dependent on the type of the argument, you'll have 2^3 functions to write. (Each of the three arguments can either be `double` or `stan::agrad::var`.) With many more arguments, this may be very difficult to maintain.

### Example with a templated function

For this example, rather than writing 8 separate functions to be efficient, we'll write a single templated function to handle all possible combinations of `double` and `stan::agrad::var`. The new piece we're introducing is the use of `stan::return_type`, a template metaprogram in Stan, that automatically determines the correct return type of the function.

(Sorry, but this won't cover how to use templates in C++.)

```
...
#include <cmath>  // std::log
#include <vector> // std::vector

template <class T1, class T2, class T3>
typename stan::return_type<T1, T2, T3>::type
foo(const T1& y, const T2& mu, const T3& sigma) {
  using std::log;
  using stan::math::square;
  using stan::math::pi;
  
  return -0.5 * square((y - mu) / sigma) - log(sigma)
    - 0.5 * log(2 * pi());
}

int main() {
  std::vector<stan::agrad::var> independent_vars;
  std::vector<double> gradients;

  double y1(0);
  stan::agrad::var mu1(0), sigma1(1);
  stan::agrad::var lp1 = foo(y1, mu1, sigma1);
  independent_vars.push_back(mu1);
  independent_vars.push_back(sigma1);

  lp1.grad(independent_vars, gradients);
  stan::agrad::recover_memory();
  std::cout << "foo(" << y1 << ", " << mu1
            << ", " << sigma1 << ") = " << lp1 << std::endl
            << "dfoo / dmu1 = " << gradients[0] << std::endl
            << "dfoo / dsigma1 = " << gradients[1] << std::endl;

  stan::agrad::var y2(1), mu2(2), sigma2(3);
  stan::agrad::var lp2 = foo(y2, mu2, sigma2);
  independent_vars.clear();
  independent_vars.push_back(y2);
  independent_vars.push_back(mu2);
  independent_vars.push_back(sigma2);

  lp2.grad(independent_vars, gradients);
  stan::agrad::recover_memory();
  std::cout << "foo(" << y2 << ", " << mu2
            << ", " << sigma2 << ") = " << lp2 << std::endl
            << "dfoo / dy2 = " << gradients[0] << std::endl
            << "dfoo / dmu2 = " << gradients[1] << std::endl
            << "dfoo / dsigma2 = " << gradients[2] << std::endl;
  return 0;
}
```

This single templated function will handle all combinations of `double` and `stan::agrad::var` and will use the efficient version of every function.


# Troubleshooting

We'll populate this section with common problems. For the time being, please join the [stan-users](https://groups.google.com/forum/?fromgroups#!forum/stan-users) email list.
