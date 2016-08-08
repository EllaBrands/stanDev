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

Depending on where you put the function, you'll need to include its definition file in the appropriate header include to make sure it's visible to models.  For instance, if the function is going into `stan/math/functions/...` then it should be included into `stan/math/functions.hpp`.

#### Unit Testing

Implement a unit test for the function which covers both the value (i.e., that it computes the right thing), the gradients of the function, and the error behavior (e.g., when the function is passed illegal values).  Looking at a unit test of a similar function will help give a sense of how to do this.  

You should check all the ways in which it can be called --- that is all permutations of scalars/vectors and double and autodiff variables.

An individual unit test, e.g., `src/test/unit-agrad-rev/functions/log_test.cpp`, can be run with `make test/unit-agrad-rev/functions/log`.

After the individual test passes, make sure that `make test-unit` and `make test-headers` both pass.

Add a test with all signatures in the form of a model in `test/gm` to make sure it's compilable from within a Stan model.

#### Model Tests

We need models in test_models instantiating all the possible ways the function can be instantiated.  Examples are in `src/test/test-models/good` for ones that pass and in `src/test/test-models/bad` for ones that don't.  Specifically, the function signature tests should go in `src/test/test-models/good/function-signatures`.  Then there are drivers for these tests in `src/test/unit/lang/parser` --- look at the ones that are there to see how to test that models don't parse or throw the right warnings when trying to parse them.

#### Documentation

Add reference documentation to the manual.  Ideally with a formula for it.  Include a reference if it's uncommon.

#### Submit

Submit a pull request on GitHub.  We'll review the code, doc and tests, perhaps iterate over fixes, verify that it passes tests, and then merge it into Stan.  


### Creating a C++ Function for Stan

This next section discusses how to get a stan function up and running at a C++ level. I briefly touch on certain mathematical and programing concepts to justify the methods recommended here. The reader will find a much more elaborate discussion on the subject in *The Stan Math Library: Reverse Mode Automatic Differentiation in C++* paper (Carpenter et al. 2015).

##### 1) Automatic Differentiation Variables 

The Hamiltonian Monte Carlo algorithm, which Stan implements, uses the gradient of the posterior distribution to efficiently explore the parameter space of model. stan/math computes the adjoint of each parameter and propagates the gradient, using `var`, a class of objects that contain the value and the adjoint of a parameter.  

I will use the term "parameter" to designate both parameters and parameter dependent variables. Data on the other hand will refer to non-parameter variables.  

For a C++ function in stan/math to run properly, any argument which could be a parameter must have a template type. Declaring it as a `var` is not enough. Stan must be able to call the function using either a `double` or a `var` variable for that argument. If an argument is guaranteed to never be a parameter -- thus always being data --, it does not need to have a template type.  

Sometimes a variable can be a function of both parameters and data. In that case, if at least one of the arguments is `var`, the returned object will be too. For example, the sum of a parameter and a datum is a parameter. We can set this condition with `boost::math::tools::promote_args`:  

```bash
 using boost::math::tools::promote_args;	
 typename promote_args<T0, T1>::type scalar;
```

The type *scalar* depends on the types of *T0* and *T1*. If at least one these is a `var`, then *scalar* will be a `var`. If they are both `double`, then *scalar* will be a double too.

`promote_args` can only have up to five arguments. For more than five arguments, we can use a nested `promote_args` object:

```bash
typename promote_args<T0, T1, T2, T3, typename promote_args<T4, T5, T6>::type>::type scalar;
```

When writing a function, we should keep in mind that the operators we use must be applicable to `var`. This holds for most of the basic math functions.


##### 2) Templates and other coding practices

As explained, functions need to be templated to allow Stan to treat parameter arguments as either a `var` or a `double`. 

The syntax for a template function could be: 

```bash
template <typename T>
T foo(T,int,int) {…}
```

which declares a function that returns an object of type *T*, and takes in three arguments, one *T* object and two integers. 

The following exemplifies what the declaration of a function in stan/math might look like. In this specific case, we are looking at a function that returns a matrix and takes four vectors as its input: 

```bash
template <typename T1, typename T2> 			
Eigen::Matrix <typename promote_args<T1, T2>::type, Dynamic, Dynamic> 
foo(const std::vector<T1>& ArrayArgument1,							
    const std::vector<T2>& ArrayArgument2,							
    const std::vector<int>& IntArrayArgument1,							
    const std::vector<int>& IntArrayArgument2) 							
```

Notice we start with a template list. The next line gives the return type: a matrix, the elements of which have a promoted type. Finally, the arguments of the function are standard vectors. The first two arguments contain template elements. We however do not use template for integers, because the Hamiltonian Monte Carlo algorithm only deals with continuous variables (therefore integers cannot be parameters!).  

There are a few other C++ characteristics in this declaration worth pointing out. First, the arguments of the function are constants, as enforced by the term `const`. This safety measure insures the original input is not modified by the function. If the programmer inadvertently modifies a constant, the compiler returns an error.  

Secondly, the `&` indicates we are using a reference, i.e we are directly accessing the memory address of the variable. If we do not use a reference, the function acts on a copy of the variable, created when the function is called. The advantage of using a reference here is that if we pass a large vector, we do not waste time and memory creating a copy of the vector the function acts on. 

There are many other ways of declaring a function in Stan. The best way to learn is to look at the many examples in the stan/math library. 

##### 3) Checking for invalid arguments and rejecting metropolis proposals

stan::math contains functions that can spot invalid arguments, thence stopping a run and producing a helpful error message. For example, one of the arguments of `integrate_ode_bdf()` is the *maximum number of steps*, which is a tuning parameter for the ODE integrator. This argument must be positive; if it isn't, the code should not run but return an informative error message. This is done with the following line of code:

```
if (max_num_steps <= 0)
        invalid_argument("integrate_ode_bdf",
                         "max_num_steps,", max_num_steps,
                         "", ", must be greater than 0");
```

which could return the error message:

```
Exception thrown at line 169: integrate_ode_bdf: max_num_steps, -100000000, must be greater than 0
```

`invalid_argument` is particularly helpful for cases where a user misspecifies the value or structure of an argument (for example, a real has the wrong sign or a matrix does not have the right dimensions). There is however no need to control whether the user passes in the right type of argument, say real versus integer. The stan parser controls this automatically. 

Another error we may want to catch could be an invalid parameter value. This is not something the user controls directly, since the value of a parameter changes as the Markov Chain explores the parameter space. In this scenario, we do not want to abort the run, but to reject the Metropolis Proposal and make the chain "turn around". To deal with this, Stan uses "check" functions. 

For example, `integrate_ode_bdf` requires the initial state to be finite:

```
stan::math::check_finite("integrate_ode_bdf", "initial state", y0);
```

which could produce the error message:

```
Informational Message: The current Metropolis proposal is about to be rejected because of the following issue:
Exception thrown at line 169: integrate_ode_bdf: initial state is inf, but must be finite! 
```

There exist a series of check functions, some of which are listed here:

|       |
| --------- |
|check_finite|
|check_positive|
|check_positive_finite |
|check_nonzero_size|
|check_ordered|

Contributors should look through existing examples to find more check functions.  


##### 4) Exposing Functions to Stan

For a function to be built into Stan, we need to take two steps: (a) Add the function to the appropriate mat header file and (b) expose the function signature in `function_signature.h` 

###### (a) The Math Header File

Functions in stan/math are located under one of five directories: fwd, memory, mix, prim, and rev. Each contains a header file `mat.hpp`. A function defined in, for example, the rev directory should be included in `stan/math/rev/mat.hpp`, using the following line of code: 

```bash
#include <stan/math/rev/foo.hpp>
```

######(b) The Function_signatures file

The `function_signatures.h` file is located under `stan/src/stan/lang`. Note that we are now no longer working on stan/math, but on the stan language itself. This means edits need to be made in the stan-dev/stan git repository. 

The `add` procedure exposes the signature of a function to stan/math. The first argument gives the name of the function, the second argument specifies the return type, and the next arguments the argument types of the function. The following line exposes a function "foo" which returns a double and takes two integers as an input:

```bash
add("foo", DOUBLE_T, INT_T, INT_T);
```

For overloaded functions, we need to add a signature for each combination of variable types. If `foo` admits either an integer or a double as its first argument, the proper way to expose the function would be:

```bash
add("foo", DOUBLE_T, INT_T, INT_T);
add("foo", DOUBLE_T, DOUBLE_T, INT_T);
```

Stan defines a few arguments that have a template type, such as `MATRIX_T` and `VECTOR_T`. For example, to expose a function that returns a templated vector:

```bash
add("foo", VECTOR_T, ...);
```

`add` can only have up to 7 arguments. Fortunately we can pass in a vector of arguments, which will only count as one argument. 

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

Once we have modified `function_signatures.h`, we need to compile stan. 


#####5) Adding Higher-Order Functions
I will elaborate more on this section later, but here's at least a good place to start at.

Higher-order functions are functions that take other functions as arguments. An example of such a function is an Ordinary Differential Equation integrator. `Add()` does not handle arguments which are function, and we therefore need to do a bit more work to expose higher-order functions to Stan. 

The first step is still to include the file that contains the function's definition in a mat header file. 

We will however not edit `function_signatures.h`, and instead directly modify Stan's grammar. This can be a rather involved process, but things tend to work out just fine if we mimic what has been done for other higher-order functions, namely `integrate_ode_rk45()` and `integrate_ode_bdf()`. Note these two functions have identical signatures, and have been added to Stan's grammar using a control structure `integrate_ode_control`. This approach saves a lot of time, because we only need to modify the grammar files once. Exposing a function directly to the Stan language requires editing the following files: 

|       |
| --------- |
|ast_def.cpp|
|ast.hpp|
|generator.hpp| 
|semantic_actions_def.cp|
|semantic_actions.hpp|
|term_grammar_def.hpp|
|term_grammar.hpp|
