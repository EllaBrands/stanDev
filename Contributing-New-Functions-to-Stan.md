Sometimes a new function is needed in Stan.  This function may compute some special mathematical function, or may be intended to speed up automatic differentiation by avoiding intermediate computations and directly computing the gradients.  This page documents the basic steps necessary to do this.

To be available as part of the Stan language, a function must be added in two places.  First, the C++ relevant code should be added to the [Stan math library](https://github.com/stan-dev/math), along with tests that demonstrate its intended behavior.  Second, the function must be added to the Stan language in the [Stan repository](https://github.com/stan-dev/stan), along with tests that demonstrate that it is parsed as intended.  If you only want a new function available through the Stan math library C++ API it does not need to be exposed to the Stan language. 

Before doing anything, please read the Developer Process document, setup your git repository and create a `feature/<new function>` branch.  The steps below should, roughly speaking, consist of one commit each in this repository.

#### Implementing the Function

Functions should be implemented in the GitHub repo stan-dev/math, not in this repo, stan-dev/stan.  That means you need a new issue for the math repo and unit tests there.  When that's all done and tested, you can come back to do the Stan pieces of the puzzle, which are straightforward.

Implement the new function in the appropriate location, most likely in the [stan/math](https://github.com/stan-dev/math) repository and include it in the appropriate header files.  The best way to figure out where this is and how to implement the function is to find a similar function already in the API and use it as a model.  You can grep to see where it's included.  

It is important to implement the function in such a way that it is templated such that it can be called with auto-diff variables for parameters and double values for data.  Again, see the existing functions for examples.

#### Directories for code

In general, implementations should go int the lowest directory possible, subject to the orderings

```
 scal < arr < mat

 prim < rev < mix;

 prim < fwd < mix;
```

If the left-hand type occurs, the directory has to be at least the minimum on the right

```
  Code includes     directory must be at least
  -------------     --------------------------
  Eigen::Matrix     mat
  std::vector       arr
  stan::math::rev   rev
  stan::math::fwd   fwd
```

Everything else follows from these two rules.  The directories are laid out two deep, with

```
(prim | fwd | rev | mix) / (scal | arr | mat)
```



#### Expose Function Signature to Stan Models

Expose the function to the parser by adding the appropriate code to [`src/stan/lang/function_signatures.hpp`](https://github.com/stan-dev/stan/blob/develop/src/stan/lang/function_signatures.h) and implement function signature tests by adding models to `src/test/gm/model_specs/compiled/`.

#### Add to Relevant Header File

Depending on where you put the function (see the rules above), you'll need to include its definition file in the appropriate header include to make sure it's visible to models.  For instance, if the function is going into `stan/math/rev/arr/foo.hpp` then `#include <stan/math/rev/arr/foo.hpp>` should be included added to `stan/math/rev/arr.hpp`.

#### Unit Testing

Implement a unit test for the function which covers both the value (i.e., that it computes the right thing), the gradients of the function, and the error behavior (e.g., when the function is passed illegal values).  Looking at a unit test of a similar function will help give a sense of how to do this.  

You should check all the ways in which it can be called --- that is all permutations of scalars/vectors and double and autodiff variables.

An individual unit test, e.g., `src/test/unit-agrad-rev/functions/log_test.cpp`, can be run with `make test/unit-agrad-rev/functions/log`.

After the individual test passes, make sure that `make test-unit` and `make test-headers` both pass.

Add a test with all signatures in the form of a model in `test/gm` to make sure it's compilable from within a Stan model.

#### Model Tests

We need models in test_models instantiating all the possible ways the function can be instantiated.  Examples are in `src/test/test-models/good` for ones that pass and in `src/test/test-models/bad` for ones that don't.  Specifically, the function signature tests should go in `src/test/test-models/good/function-signatures`.  Then there are drivers for these tests in `src/test/unit/lang/parser` --- look at the ones that are in `src/test/unit/lang/parser/other_test.cpp` to see how to test that models don't parse or throw the right warnings when trying to parse them.

#### Documentation

Add reference documentation to the manual.  Ideally with a formula for it.  Include a reference if it's uncommon.

#### Submit

Submit a pull request on GitHub.  We'll review the code, doc and tests, perhaps iterate over fixes, verify that it passes tests, and then merge it into Stan.  


### Creating a C++ Function for Stan

This next section discusses how to get a stan function up and running at a C++ level. 

##### 1) Parameter Variables 

I will use the term "parameter" to designate parameters and parameter dependent variables. Data on the other hand will refer to non-parameter variables.

Stan computes the gradient of the posterior to efficiently explore the parameter space of a model. For each parameter, stan stores the value and the derivative in a object `var` or `fvar`, respectively used for reverse and forward mode automatic differentiation.  

A function acting on a parameter should be overloaded for `double`, `var`, and `fvar` types. The easiest way to do this is to use a template type. If you decide to code the derivatives by hand, you need to overload the function. For example *sin(x)* has the signatures `sin(var x)` and `sin(fvar<T> x)`, and the derivative is explicitly coded as *cos(x)*. If you do not explicitly code the derivative, automatic differentiation *approximates* the derivative. 

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

There exist a bunch of check functions. Here is a (far from complete) list:

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

######(b) The Function_signatures file

The `function_signatures.h` file is located under `stan/src/stan/lang`. Note we are now no longer working in stan/math, but in stan. This means you need to edit the stan-dev/stan git repository. 

The `add` procedure exposes the signature of a function to stan. The first argument gives the name of the function, the second argument specifies the return type, and the next arguments the argument types of the function. The following line exposes a function "foo" which returns a double and takes in two integers:

```bash
add("foo", DOUBLE_T, INT_T, INT_T);
```

For overloaded functions, we need to add a signature for each combination of variable types. If `foo` admits either an integer or a double as its first argument, the proper way to expose the function would be:

```bash
add("foo", DOUBLE_T, INT_T, INT_T);
add("foo", DOUBLE_T, DOUBLE_T, INT_T);
```

Stan defines a few arguments that have a template type, such as `MATRIX_T` and `VECTOR_T`. For example, to expose a function that returns a template vector:

```bash
add("foo", VECTOR_T, ...);
```

`add` can only have up to 7 arguments. Fortunately you can pass in a vector of arguments, which will only count as one argument. 

The following lines expose the foo function used as an example in the previous section:

```bash
std::vector<expr_type> arg_types; // declare a vector, which contains elements of type "arg_types"
// Next, add elements to our vector
for(int i = 0; i<2; i++){arg_types.push_back(vector_types[1]);}
for(int i = 0; i<2; i++) {arg_types.push_back(int_vector_types[1]);}
add("foo", MATRIX_T, arg_types);
```

The object `vector_types[1]` designates a standard vector (`std::vector`) of template elements, and `int_vector_types[1]` refers to a standard vector of integer.  These objects are defined at the top of `function_signatures.h`.   

It is important to understand what these variables correspond to in Stan, in the signature file, and in C++. 


| Stan  | ADD | C++                        |
| ------------- | ------------- | ----------------------------------- |                       
| vector  | VECTOR_T  | eigen::vector<T>   |
| matrix  | MATRIX_T  | eigen::Matrix<T, Eigen::Dynamic, Eigen::Dynamic>   |
| array[real] | vector_types[1] | std::vector<T> |
| array[int] | int_vector_types[1] | std::vector<int> | 


`T`, as usual, is a template. 

Stan uses two types of vectors in C++. Standard vectors (`std::vector`) are computationally more efficient for index operations (i.e going through each element of the vector one by one). Eigen vectors (`Eigen::vector`) on the other hand are faster when doing vector and matrix operations.   

Once you've modified `function_signatures.h`, we need to compile stan. 


#####5) Adding Higher-Order Functions
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