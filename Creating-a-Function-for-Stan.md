## Creating a C++ Function for Stan

This page complements the **Contributing New Functions in Stan** page, and discusses how to get a function up and running at a C++ coding level. We only briefly touch on certain mathematical and programing concepts to justify the methods recommended here. The reader will find a much more elaborate discussion of these concepts in *The Stan Math Library: Reverse Mode Automatic Differentiation in C++* paper (Carpenter et al. 2015).


### Automatic Differentiation Variables
The Hamiltonian Monte Carlo algorithm, which Stan implements, uses the gradient of the posterior distribution to efficiently explore the parameter space. stan/math computes the adjoint of each parameter and propagates the gradient, using `var`, a class of objects that contain the value and the adjoint of a parameter.  

We will use the term "parameter" to designate both parameters and parameter dependent variables. Data on the other hand will refer to non-parameter variables.  

For a C++ function in stan/math to run properly, any argument which could be a parameter must have a template type. Declaring it as a `var` is not enough. Stan must be able to call the function using either a `double` or a `var` variable for that argument. If an argument is guaranteed to never be a parameter -- thus being data --, it does not need to have a template type.  

Sometimes a variable can be a function of both parameters and data. In that case, if at least one of the arguments is `var`, then the returned object will be too. For example, the sum of a parameter and a datum is a parameter. We can set this condition with `boost::math::tools::promote_args`:  

```bash
 using boost::math::tools::promote_args;	
 typename promote_args<T0, T1, T2, T3, T4>::type scalar;
```

The type *scalar* depends on the types of *T0*, *T1*, *T2*, *T3* and *T4*. If at least one these is a `var`, then *scalar* will be a `var`. If they are all `double`, then *scalar* will be too.

`promote_args` can only have up to five arguments. For more than five arguments, we can use a nested `promote_args` object:

```bash
typename promote_args<T0, T1, T2, T3, typename promote_args<T4, T5, T6>::type>::type scalar;
```

When writing a function, we should keep in mind that the operators we use must be applicable to `var`. This holds for most of the basic math functions.
	
That’s it! The rest is regular C++ with the caveat that every function and class must be templated if they contain parameters.

#### C++ Coding: Templates and other practices

As explained in the previous section, functions need to be templated to allow Stan to treat parameter arguments as either a `var` or a `double`. 

The syntax for a template function could be: 

```bash
template <typename T>
T foo(T,int,int) {…}
```

which gives us a function that returns an object of type *T*, and takes in three arguments, one *T* object and two integers. 

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

There are a few other C++ characteristics in this declaration worth pointing out: 

##### Constants

The arguments of the function are constants, as prescribed by the term `const`. This safety measure insures the original input is not modified by the function. If the programmer inadvertently modifies a constant, the compiler returns an error.  

##### References

The `&` indicates we are using a reference, i.e we are directly accessing the memory address of the variable. If we do not use a reference, the function acts on a copy of the variable, created when the function is called. 

The advantage of using a reference here is that if we pass a large vector, we do not waste time and memory creating a copy of the vector the function acts on. 

There are many other ways of declaring a function in Stan, and the best way to learn is to look at the many examples in the stan/math library. 


### Exposing Functions to Stan/math

For a function to be built into Stan, we need to take two steps: (a) Add the function to the `math.hpp` file and (b) expose the function signature in `function_signature.h` 

####(a) The Math File

The math file can be found under `stan/lib/stan/math`. `math.hpp` is a header file which stores the address of other header files, which in turn include the files in which functions are defined. We'll have to add the address of the file in which our function is defined. For example, for a function defined in a file `foo.hpp` under the `rev` directory:  
```bash
#include <stan/math/rev/foo.hpp>
```
[NOTE: I noticed the way header files were organized has recently changed, probably need to specify how to navigate the various header files]


####(b) The Function_signatures file

The `function_signatures.h` file is located under `stan/src/stan/lang`. 

The `add` function exposes the signature of a function to stan/math. The first argument gives the name of the function, the second argument specifies the return type, and the next arguments the argument types of the function. The following line exposes a function "foo" which returns a double and takes two integers as an input:

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

The `add` function can only have up to 7 arguments. Fortunately we can pass in a vector of arguments, which will only count as one argument. 

The following lines expose the foo function used as an example in the previous section:

```bash
std::vector<expr_type> arg_types; // declare a vector, which contains elements of type "arg_types"
// Next, add elements to our vector
for(int i = 0; i<2; i++){arg_types.push_back(vector_types[1]);}
for(int i = 0; i<2; i++) {arg_types.push_back(int_vector_types[1]);}
add("foo", MATRIX_T, arg_types);
```
Had we not use a vector that stores the arguments of foo, `Add()` would have had more than 7 arguments. The object `vector_types[1]` designates a standard vector (`std::vector`) of template elements, and `int_vector_types[1]` refers to a standard vector of integer.  These objects are defined at the top of `function_signatures.h`.   

It is important to understand what these variables correspond to in Stan, in the signature file, and in C++. 


| Stan  | ADD | C++                        |
| ------------- | ------------- | ----------------------------------- |                       
| vector  | `VECTOR_T`  | `eigen::vector<T>`   |
| matrix  | `MATRIX_T`  | `eigen::Matrix<T, Eigen::Dynamic, Eigen::Dynamic>`   |
| array[real] | `vector_types[1]` | `std::vector<T>` |
| array[int] | `int_vector_types[1]` | `std::vector<int>` | 


`T`, as usual, is a template. 

Stan uses two types of vectors in C++. Standard vectors (`std::vector`) are computationally more efficient for index operations (i.e going through each element of the vector one by one). Eigen vectors (`Eigen::vector`) on the other hand are faster when doing vector and matrix operations.   

Once we have modified `function_signatures.h`, we need to compile stan/math. 


###4. Adding Higher-Order Functions
I will elaborate more on this section later, but here's at least a good place to start at.

Higher-order functions are functions that take other functions as arguments. An example of such a function would be an Ordinary Differential Equation integrator. `Add()` does not handle arguments which are function, and we therefore need to do a bit more work to expose higher-order functions to Stan. 

The first step is to, once again, include (directly or indirectly) the file that contains the function's definition in `math.hpp`. 

We will however not add the function's signature to `function_signatures.h`, and instead directly modify Stan's grammar. This can be a rather involved process, but things tend to work out just fine if we mimic what has been done for other higher-order functions, such as `integrate_ode()` and `integrate_ode_cvode()`. Exposing a function directly to the Stan language requires modifying the following files: `ast_def.cpp`, `ast.hpp`, `generator.hpp`, `limits.hpp`, `semantic_actions_def.cpp`, `semantic_actions.hpp`, `term_grammar_def.hpp`, and `term_grammar.hpp`.