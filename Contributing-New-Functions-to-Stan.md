Sometimes a new function is needed in Stan.  This function may compute some special mathematical function, or may be intended to speed up automatic differentiation by avoiding intermediate computations and directly computing the gradients.  This page documents the basic steps necessary to do this.

To be available as part of the Stan language, a function must be added in two places.  First, the C++ relevant code should be added to the [Stan math library](https://github.com/stan-dev/math), along with tests that demonstrate its intended behavior.  Second, the function must be added to the Stan language in the [Stan repository](https://github.com/stan-dev/stan), along with tests that demonstrate that it is parsed as intended.  If you only want a new function available through the Stan math library C++ API it does not need to be exposed to the Stan language. 

Before doing anything, please read the [Stan Developer Process](https://github.com/stan-dev/stan/wiki/Developer-process-overview) document, setup your git repository and create a `feature/<new function>` branch.  The steps below should, roughly speaking, consist of one commit each in this repository.

#### Implementing the Function

Functions should be implemented in the GitHub repo stan-dev/math, not in this repo, stan-dev/stan.  That means you need a new issue for the math repo and unit tests there.  When that's all done and tested, you can come back to do the Stan pieces of the puzzle, which are straightforward.

Implement the new function in the appropriate location, most likely in the [stan/math](https://github.com/stan-dev/math) repository and include it in the appropriate header files.  The best way to figure out where this is and how to implement the function is to find a similar function already in the API and use it as a model.  You can grep to see where it's included.  

It is important to implement the function in such a way that it is templated such that it can be called with auto-diff variables for parameters and double values for data.  Again, see the existing functions for examples.

#### Directories for code (Math library)

In general, implementations should go int the lowest directory possible, subject to the orderings

```
 scal < arr < mat

 prim < rev < mix;

 prim < fwd < mix;
```

If the left-hand type occurs, the directory has to be at least the minimum on the right

```
  Code includes      directory must be at least
  -------------      --------------------------
  Eigen::Matrix      mat
  std::vector        arr
  stan::math::var    rev
  stan::math::fvar   fwd
```

Everything else follows from these two rules.  The directories are laid out two deep, with

```
(prim | fwd | rev | mix) / (scal | arr | mat)
```

#### Add to Relevant Header File (Math library)

Depending on where you put the function (see the rules above), you'll need to include its definition file in the appropriate header include to make sure it's visible to models.  For instance, if the function is going into `stan/math/rev/arr/foo.hpp` then `#include <stan/math/rev/arr/foo.hpp>` should be included added to `stan/math/rev/arr.hpp`.

#### Unit Testing (Math library)

Implement a unit test for the function which covers both the value (i.e., that it computes the right thing), the gradients of the function, and the error behavior (e.g., when the function is passed illegal values).  Looking at a unit test of a similar function will help give a sense of how to do this.  

You should check all the ways in which it can be called --- that is all permutations of scalars/vectors and double and autodiff variables.

An individual unit test, e.g., [`test/unit/math/prim/scal/fun/abs_test.cpp`](https://github.com/stan-dev/math/blob/develop/test/unit/math/prim/scal/fun/abs_test.cpp), can be run with `./runTests.py test/unit/math/prim/scal/fun/abs_test.cpp`.

After the individual test passes, make sure that `./runTests.py test/unit` and `make test-headers` both pass.

#### Expose Function Signature to Stan Models (Stan library)

Expose the function to the parser by adding the appropriate code to [`src/stan/lang/function_signatures.hpp`](https://github.com/stan-dev/stan/blob/develop/src/stan/lang/function_signatures.h).

#### Model Tests (Stan library)

We need models in test_models instantiating all the possible ways the function can be instantiated.  Examples are in [`src/test/test-models/good`](https://github.com/stan-dev/stan/blob/develop/src/test/test-models/good) for ones that pass and in [`src/test/test-models/bad`](https://github.com/stan-dev/stan/blob/develop/src/test/test-models/bad) for ones that don't. Specifically, the function signature tests should go in [`src/test/test-models/good/function-signatures`](https://github.com/stan-dev/stan/blob/develop/src/test/test-models/good/function-signatures).

There are drivers for these tests in [`src/test/unit/lang/parser`](https://github.com/stan-dev/stan/blob/develop/src/test/unit/lang/parser) --- look at the ones that are in [`src/test/unit/lang/parser/other_test.cpp`](https://github.com/stan-dev/stan/blob/develop/src/test/unit/lang/parser/other_test.cpp) to see how to test that models don't parse or throw the right warnings when trying to parse them.

#### Documentation (Math and Stan libraries)

Add Doxygen docs to any new functions and reference documentation to the manual (in the Stan library).  Ideally with a formula for it.  Include a reference if it's uncommon.

#### Submit

Submit a pull request on GitHub.  We'll review the code, doc and tests, perhaps iterate over fixes, verify that it passes tests, and then merge it into Stan. Details on the submission process can be found on the [Git-Process](https://github.com/stan-dev/stan/wiki/Dev:-Git-Process) wiki page.

### Creating a C++ Function for Stan

This next section discusses how to get a stan function up and running at a C++ level. 

##### 1) Parameter Variables 

I will use the term "parameter" to designate parameters and parameter dependent variables. Data on the other hand will refer to non-parameter variables.

Stan computes the gradient of the posterior to efficiently explore the parameter space of a model. For each parameter, stan stores the value and the derivative in a object `var` or `fvar`, respectively used for reverse and forward mode automatic differentiation.  

A function acting on a parameter should be overloaded for `double`, `var`, and `fvar` types. The easiest way to do this is to use a template type. If you decide to code the derivatives by hand, you need to overload the function. For example *sin(x)* has the signatures `sin(double)`, `sin(const var& x)` and `template <typename T> sin(const fvar<T>& x)`;  in the latter two, the partial derivatives are explicitly coded. If you do not explicitly code the derivative, automatic differentiation *approximates* the derivative. 

A variable may be a function of parameters and data. In that case, if at least one of the arguments is a parameter, so is the returned object. This is consitent with the above definition of a parameter.

For example, the sum of a parameter and a datum is a parameter. 

We can set this condition using `boost::math::tools::promote_args`:  

```bash
 using boost::math::tools::promote_args;	
 typename promote_args<T0, T1>::type scalar;
```

The type *scalar* depends on the types  *T0* and *T1*. If at least one these is a parameter, then *scalar* is a parameter. If they are both `double`, then *scalar* is a double.

`promote_args` can only have up to five arguments. For more than five arguments, we can use a nested `promote_args` object:

```bash
typename promote_args<T0, T1, T2, T3, typename promote_args<T4, T5, T6>::type>::type scalar;
```

When writing a function, you should keep in mind  the operators you use must apply to `var` and `fvar`. This holds for most basic math functions.


##### 2) C++ Coding: Templates, Constant Arguments, and References

There are few common C++ practices in Stan worth pointing out.

Consider the following lines of code. In this particular example, a *foo* returns a matrix and takes in four vectors: 

```bash
template <typename T1, typename T2>
Eigen::Matrix <typename promote_args<T1, T2>::type, Dynamic, Dynamic>
foo(const std::vector<T1>& ArrayArgument1,
    const std::vector<T2>& ArrayArgument2,
    const std::vector<int>& IntArrayArgument1,
    const std::vector<int>& IntArrayArgument2)
```

We start with a template list. The next line gives the return type: a matrix, the elements of which have a promoted type. The arguments are standard vectors. The first two arguments contain template elements. We however do not use a template type for integers, because Stan only deals with continuous variables (integers cannot be parameters!).  

Next, notice the arguments of the function are constants, as enforced by the term `const`. This safety measure insures the original input is not modified by the function. If the programmer inadvertently modifies a constant, the compiler returns an error.  

Secondly, the `&` indicates we are using a reference: we are directly accessing the memory address of the variable. If we do not use a reference, the function acts on a copy of the variable, created when the function is called. With a reference, we do not waste time and memory creating a copy of the vector the function acts on. 

There are many other ways of declaring a function in Stan. The best way to learn is to look at the many examples in stan/math. 

##### 3) Checking for Invalid Arguments and Rejecting Metropolis Proposals

A function should spot invalid arguments and produce helpful error messages. For example: one of the arguments of `integrate_ode_bdf()` is the *maximum number of steps*. This argument needs to be positive; if it isn't, the code does not run but returns an informative error message. We get this behavior with the following code:

```
if (max_num_steps <= 0)
        invalid_argument("integrate_ode_bdf",
                         "max_num_steps,", max_num_steps,
                         "", ", must be greater than 0");
```

If I use $maximum number of steps = -1$, the error message is:

```
Exception thrown at line 169: integrate_ode_bdf: max_num_steps, -1, must be greater than 0
```

Checks for invalid arguments are helpful when a user misspecifies the value or structure of an argument (for example, a real has the wrong sign or a matrix does not have the right dimensions). There is however no need to control whether the user passes the right type of argument, say real versus integer. The stan parser controls this automatically. 

Another error type we want to catch is invalid parameter values. This is not something the user controls directly, because the value of a parameter changes as the Markov Chain explores the parameter space. Our goal this time is not to abort the run, but to reject the Metropolis Proposal and make the chain "turn around". For this purpose, Stan uses "check" functions. 

For example, `integrate_ode_bdf` requires the initial state to be finite:

```
stan::math::check_finite("integrate_ode_bdf", "initial state", y0);
```

If the initial state is infinite, the code produces the error message:

```
Informational Message: The current Metropolis proposal is about to be rejected because of the following issue:
Exception thrown at line 169: integrate_ode_bdf: initial state is inf, but must be finite! 
```

There are many check functions. Here is a (far from complete) list:

|       |
| --------- |
|check_finite|
|check_positive|
|check_positive_finite |
|check_nonzero_size|
|check_ordered|


Contributors should look through existing examples to find more check functions.  


##### 4) Exposing Functions to Stan

For a function to be built into Stan, we need to: (a) include the function in the appropriate header file and (b) add the function signature in `function_signature.h` 

###### (a) Header Files

Functions in stan/math are located under one of five directories: fwd, memory, mix, prim, and rev, which in turn contain the directories: mat, arr, and scal, with a corresponding header file for each.   

A function defined in the rev/mat directory should be included in `stan/math/rev/mat.hpp`, using the following line of code: 

```bash
#include <stan/math/rev/foo.hpp>
```

###### (b) The file `function_signatures.h` for the Stan language compiler `stanc`

The `function_signatures.h` file is located under `stan/src/stan/lang`. Note we are now no longer working in stan/math, but in stan. This means you need to edit the stan-dev/stan git repository. 

This file is used by the Stan language parser.  It holds the intermediate representation of a function call in a Stan program in terms of the parser's abstract syntax tree (AST) types.  For the parser, non-array variables are of class `bare_expr_type` and array variables are of class `bare_array_type`.  

The `add` procedure exposes the signature of a function to the parser. The first argument gives the name of the function, the second argument specifies the return type, and the next arguments the argument types of the function. The following line exposes a function "foo" which returns a double and takes in two integers:

```bash
add("foo", bare_expr_type(double_type()), bare_expr_type(int_type()), bare_expr_type(int_type()));
```

For overloaded functions, we need to add a signature for each combination of variable types. If `foo` admits either an integer or a double as its first argument, the proper way to expose the function would be:

```bash
add("foo", bare_expr_type(double_type()), bare_expr_type(int_type()), bare_expr_type(int_type()));
add("foo", bare_expr_type(double_type()), bare_expr_type(double_type()), bare_expr_type(int_type()));
```

It is important to understand the correspondence between the Stan language types, the Stan parser AST types used in the file `function_signatures.h`, and the type in the generated C++ code

| Stan  | function_signatures.h | C++                        |
| ------------- | ------------- | ----------------------------------- |                       
| vector  | bare_expr_type(vector_type())  | eigen::vector&lt;T&gt;   |
| matrix  | bare_expr_type(matrix_type())  | eigen::Matrix&lt;T, Eigen::Dynamic, Eigen::Dynamic&gt;   |
| array of reals | bare_expr_type(bare_array_type(double_type(), 1) | std::vector&lt;T&gt; |
| array of ints | bare_expr_type(bare_array_type(int_type(), 1) | std::vector&lt;int&gt; | 


`T`, as usual, is a template. 

Stan uses two types of vectors in C++. Standard vectors (`std::vector`) are computationally more efficient for index operations (i.e going through each element of the vector one by one). Eigen vectors (`Eigen::vector`) on the other hand are faster when doing vector and matrix operations.   

Once you've modified `function_signatures.h`, we need to compile stan. 


####  Adding Higher-Order Functions

This is a little more specialized. As of now, there is no simple mechanism to add higher-order functions to Stan. Here's at least a good place to start.

Higher-order functions are functions that take other functions as arguments. An example of such a function is an Ordinary Differential Equation integrator. `Add()` does not handle arguments which are function, so we need to do a bit more work to expose higher-order functions to Stan. 

The work we do in stan/math remains identical. The work in stan is different.

We do not edit `function_signatures.h`, but instead "directly" modify Stan's grammar. This can be a rather involved process, but things tend to work out just fine if we mimic what was done for other higher-order functions, namely `integrate_ode_rk45()` and `integrate_ode_bdf()`. These two functions have identical signatures, and were added to Stan's grammar using a control structure `integrate_ode_control`. This approach saves a lot of time, because we only need to modify the grammar files once. Exposing a function directly to the Stan language requires editing the following files: 

|       |
| --------- |
|ast_def.cpp|
|ast.hpp|
|generator.hpp| 
|semantic_actions_def.cp|
|semantic_actions.hpp|
|term_grammar_def.hpp|
|term_grammar.hpp|

#### Adding Gaussian Process Covariance Functions

Implementing Gaussian processes requires three different functions.  Details are given in https://github.com/stan-dev/stan/wiki/Adding-a-Gaussian-Process-Covariance-Function. 


## Testing New Functions


There are a mish-mash of testing frameworks that various developers have sprinkled throughout the `unit/test` directory of the `stan-dev/math` repo.  This summarizes what is going on.  It should make writing tests for functions easier.

#### Testing simple functions

For testing one and two scalar argument functions, there are two alternatives, depending on whether the function i vectorized or not.

For the simple unvectorized tests, you can see an example of a complete test for operator addition here:

```
test/unit/math/mix/core/operator_addition_test.cpp
```

You will see that it defines a structure 

```
struct op_addition_f {
  template <typename T1, typename T2>
  static typename boost::math::tools::promote_args<T1, T2>::type
  apply(const T1& x1, const T2& x2) {
    return x1 + x2;
  }
};
```

with a static tempate function `apply()`, which just passes through the result to `operator+()`.  Autodiff versions will be brought in by argument-dependent lookup. 

The test itself is then run with:

```
TEST(mathMixCore, opratorAddition) {
  stan::math::test::test_common_args<op_addition_f, false>();
}
```

The first template parameter is the name of the structure defining the function, here `op_addition_f`.  The second argument is `true` if the function is a comparison operator (they return integer values, not real values, so are tested slightly differently).  This function then cycles through various typical arguments, positive, zero, negative, infinite, and not-a-number, making sure that the result of the basic function on `double` values matches and the derivatives match at all levels of autodiff.

The actual code for testing each level of autodiff is in the framework file:
```
test/unit/math/mix/mat/util/autodiff_tester.hpp
```

The tests cover all the autodiff cases.

* reverse: `var`, 
* forward: `fvar<double>`, `fvar<fvar<double>>`
* mixed: `fvar<var>`, `fvar<fvar<var>>`

**Warning:** The test framework assumes you already have tests for primitives, `double` and `int`.

#### Testing vectorized functions

Follow the example in `test/unit/math/mix/mat/fun/acosh_test.cpp`.  

You'll see that there's a required static `apply` function, as before, that wraps up the `acosh()` function and has a `using stan::math::acosh` file to allow it to be found along with the `std::` namespace version.

There is then an integer and templated `apply_base` function which just call the static function above.  The reason `int` is broken out is that it uses the template argument of the `apply` function called to explicitly promote to `double`.  

The rest of the static functiosn provide definitions of valid and invalid inputs of both `double` and `int` varieties.

Then there are four macro calls at the end of the file to test at each level.  These get played out in test harnesses in each of the included files with the actual tests, such as

```
test/unit/math/rev/mat/vectorize/rev_scalar_unary_test.hpp
```

If you dive into that file, it's just more includes of files starting with `expect_` (named following google test conventions) that instantiate the test macros and register them with a testing harness.