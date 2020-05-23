## Testing Stan using Gnu Make and Python

The Stan repository contains functional and unit tests under the directory `src/test`.  There is a makefile for Gnu make that runs these targets, `makefile`, and a helper Python script, `runTests.py`, which calls this makefile to build and run the targets.

1. Prerequisite tools and settings.
2. How to run tests via script `runTests.py`.
3. How to run a single model integration test.

### 1. Prerequisite tools and settings.

The following programs and tools are required:
 - a C++ compiler, either g++ or clang++
 - Gnu make, version 3.8 ([http://www.gnu.org/software/make/])
 - Python:  versions 2.7 and up
 - Unix utilities `find` and `grep`

To customize Stan's build options, variables can be set in a file called `make/local`. This is the preferred way of customizing the build. You can also set it more permanently by setting variables in `~/.config/stan/make.local`.

Note: variables in `make/local` override variables in `~/.config/stan/make/local`.

**Nota bene:** these build instructions are not in the released version yet. This is for the `develop` branch (post v2.18.0. Prior to this, the build instructions are similar, but not identical.

There are a lot of make variables that can be set. In general, `CXXFLAGS_*` is for C++ compiler flags, `CPPFLAGS_*` is for C preprocessor flags, `LDFLAGS_*` is for linker flags, and `LDLBIS_*` is for libraries that need to be linked in.

These are the more common make flags that could be set:
- `CXX`: C++ compiler
- `MATH`: location of the Stan Math library.
- `CXXFLAGS_OS`: compiler flags specific to the operating system.
- `CPPFLAGS_OS`: C preprocessor flags specific to the operating system.
- `O`: optimization level. Defaults to `3`.
- `O_STANC`: optimization level for building the Stan compiler. Defaults to `0`.
- `INC_FIRST`: this is a C++ compiler option. If you need to include any headers before Math's headers, this is where to specify it. Default is empty.
- `LDFLAGS_OS`: linker flags for the operating system
- `LDLIBS_OS`: link libraries for the operating system

For additional make options, see [Building and Running Tests](https://github.com/stan-dev/math/wiki/Developer-Doc#building-and-running-tests) in the Math wiki.

The `make/local` file in the Stan repository is empty. To add `CXX=clang++` and `O=0` to `make/local` type:
```
> echo "CXX=clang++" >> make/local
> echo "O=0" >> make/local
```

### 2. Header tests

To run a single header test, add "-test" to the end of the file name. Example: 
```
> make src/stan/math/constants.hpp-test
```

Test all source headers to ensure they are compilable and include enough header files:
```
> make test-headers
```

### 3. Cpplint

Run cpplint.py on source files (unfortunately requires python 2.7)
```
> make cpplint
```

### 4. The Python script, `runTests.py`

The `runTests.py` Python script is responsible for:
 1. building the test executables using Gnu make
 2. running the test executables

Test executables will be rebuilt if any changes are made to dependencies of the executable.

Running the script without any arguments brings up some help. This is the current help message:
```
> ./runTests.py
usage: ./runTests.py <path/test/dir(/files)>
or
       ./runTests.py -j<#cores> <path/test/dir(/files)>
```

Arguments:
- the first optional argument is `-j<#cores>` for building the test in parallel.  If this argument is present, it will be passed along to the <code>make</code> command and will override system defaults and `MAKEFLAGS` set in <code>make/local</code>.
- the rest of the arguments should either specify a single test file (`*_test.cpp`) or a test directory.

#### Example: running a single unit test.

To run a single unit test, specify the path of the test as an argument. Example:
```
> ./runTests.py src/test/unit/math/prim/scal/fun/abs_test.cpp
```

To run multiple unit tests at once, specify the paths of the tests separated by a space. Example:
```
> ./runTests.py src/test/unit/math/prim/scal/fun/abs_test.cpp  src/test/unit/math/prim/scal/fun/Phi_test.cpp
```

These can also take the `-j` flag for parallel builds. For example:
```
> ./runTests.py -j4 src/test/unit/math/prim/scal/fun/abs_test.cpp
```

**Note:** On some systems it may be necessary to change the access permissions to make `runTests.py` executable:
   `> chmod +x runTests.Py`
This only needs to be done once.

#### Example: running a directory of tests.

To run a directory of tests (recursive), specify the directory as an argument. Note: there is no trailing slash (`/`). Example:
```
> ./runTests.py src/test/unit/math/prim
```

To run multiple directories, separate the directories by a space. Example:
```
> ./runTests.py src/test/unit/math/prim src/test/unit/math/rev
```

To build in parallel, provide the `-j` flag with the number of cores. Example:
```
> ./runTests.py -j4 src/test/unit/math
```


### 5. How to run a single model integration test

We have test Stan programs located in the `src/test/test-models/good` folder. For example, `src/test/test-models/good/funs1.stan`. We test that each of these can be generated into valid C++ header files that can be compiled.

The `src/test/integration/compile_models_test.cpp` will test all Stan programs in `src/test/test-models/good` are compilable.

We can also test that a single Stan program is valid. Let's say you wanted to test this Stan file: `src/test/test-models/good/map_rect.stan`

If you type:

```
> make test/test-models/good/map_rect.hpp-test
```

then it'll do a number of things (in this order):

1. build `test/test-models/stanc` --- it's a stripped down version of the compiler

2. generate the C++: it goes from `src/test/test-models/good/map_rect.stan` to `test/test-models/good/map_rect.hpp`

3. the `*-test` target attempts to compile it by including it into a file with a `main()`. That file with the `main()` looks like this (it instantiates the log prob with the types we care about):

```
// test/test-model-main.cpp
int main() {
  stan::io::var_context *data = NULL;
  stan_model model(*data);

  std::vector<stan::math::var> params;
  std::vector<double> x_r;
  std::vector<int> x_i;

  model.log_prob<false, false>(params, x_i);
  model.log_prob<false, true>(params, x_i);
  model.log_prob<true, false>(params, x_i);
  model.log_prob<true, true>(params, x_i);
  model.log_prob<false, false>(x_r, x_i);
  model.log_prob<false, true>(x_r, x_i);
  model.log_prob<true, false>(x_r, x_i);
  model.log_prob<true, true>(x_r, x_i);

  return 0;
}
```


## GoogleTest Unit Tests for C++

#### What to Test

1. We're trying to make sure all the boundary conditions get called

    * zero-size inputs for containers
    * 0, NaN, inf, and -inf inputs for floating point

2. We want to make sure all tests get tickled and the appropriate exception types are thrown and documented.  

3. We want to test that autodiff works

    * reverse-mode
    * forward mode: double (1st order), var (2nd order), fvar<var> (3rd order)

There should be a finite difference testing framework soon.  An alternative is to define a second, simpler templated versio of a function and test against that.

#### Using GoogleTest

The place to start for doc for GoogleTest (other than looking at our existing examples) are:

* [Primer](https://code.google.com/p/googletest/wiki/Primer): an overview

* [Advance Guide](https://code.google.com/p/googletest/wiki/AdvancedGuide): good description of available tests and best API guide I know

#### Common test types

The documentation (above) is excellent so check there for details and the complete list of tests.  The following is a small sample of commonly used test types in Stan and links to the relevant sections of the GoogleTest documentation.

* [EXPECT/ASSERT](https://code.google.com/p/googletest/wiki/Primer#Assertions) EXPECT records the result but allows further tests in the same function, ASSERT exits the current function without cleanup. Stan unit tests use both forms.
* [EXPECT_TRUE](https://code.google.com/p/googletest/wiki/Primer#Basic_Assertions) EXPECT_TRUE/FALSE test that the condition is TRUE/FALSE.
* [EXPECT_EQ](https://code.google.com/p/googletest/wiki/Primer#Binary_Comparison) Tests that a calculated value equals (==) an expected result exactly, not appropriate for floating point comparison.
* [EXPECT_FLOAT_EQ](https://code.google.com/p/googletest/wiki/AdvancedGuide#Floating-Point_Comparison) Tests that a floating point calculation results to known output values.
* [EXPECT_NEAR](https://code.google.com/p/googletest/wiki/AdvancedGuide#Floating-Point_Macros) Like EXPECT_FLOAT_EQ but a third argument sets the threshold used. 
* [ASSERT_THROW](https://code.google.com/p/googletest/wiki/AdvancedGuide#Exception_Assertions) Tests that a specific exception type is thrown.
* [ASSERT_NO_THROW](https://code.google.com/p/googletest/wiki/AdvancedGuide#Exception_Assertions) Tests that the statement does not throw. Some parts of the code are not supposed to throw... not sure which.

## C++ Testing Framework

### Current state of non-probability function math unit tests
In order to cut down on the inconsistencies in stan/math non-probability unit tests and the menial labor that goes into writing the unit tests, we need to design a unified testing framework.

Right now we leave unit test writing to the developer. The developer must manually instantiate all combinations of admissible types for each function written. Next, it is up to the developer to write input tests, Nan tests, valid values, etc.

#### Current state of probability function math unit tests

While we do have a framework for probability tests, the current code isn't testing second and third derivatives as was intended.

### Goal of testing framework

Generalize the current probability function testing framework so it can be used for math unit tests. The framework will implement the following tests:

* nan tests
* invalid inputs
* gradient tests
* hessian tests
* gradient of hessian tests
* values

#### Implementation details

Bob's first cut at a recursive template metaprogramming approach to automatically expanding the type combinations is pasted below in sections. The idea is that we'll move to a more general version of Nomad's functor-based testing suite. Instead of requiring explicitly written functors for each combination of types and test, we'll require a more generally written functor like so:

```
struct multiply_f {
  template <typename T1, typename T2>
  typename promote_args<T1,T2>::type
  operator()(const cons<T1, cons<T2,nil> >& x) const {
    return multiply_base(x.h_, x.t_.h_);
  }
};
```

The supporting code is below, along with the TEST construction.


Supporting cons struct required for implementation:

```
#include <gtest/gtest.h>
#include <stan/agrad/rev.hpp>
#include <boost/math/tools/promotion.hpp>
#include <iostream>
#include <vector>

namespace stan {
  namespace test {

    struct nil {
      nil() { }
    };


    template <typename H, typename T>
    struct cons {
      const H& h_;
      const T& t_;
      cons(const H& h, const T& t)
	: h_(h), 
	  t_(t)
      { }
    };

    // cons factory
    const nil& cf() {
      static const nil n;
      return n;
    }
    template <typename H>
    cons<H,nil>
    cf(const H& h) {
      return cons<H,nil>(h,cf());
    }
    template <typename H, typename T>
    cons<H,T>
    cf(const H& h, const T& t) {
      return cons<H,T>(h,t);
    }


    template <typename F, typename T>
    struct bind_head_var {
      const F& f_;
      const size_t n_;
      bind_head_var(const F& f, size_t n)
	: f_(f), 
	  n_(n)
      { }

      // T2 can't just be T because remaining types change
      template <typename T2>
      stan::agrad::var
      operator()(const T2& t,
		 std::vector<stan::agrad::var>& xs) const {
	using stan::agrad::var;
	cons<var,T2> c(xs[n_],t);
	return f_(c,xs);
      }

    };


    template <typename F, typename T>
    struct bind_head_dbl {
      const F& f_;
      const double x_;
      bind_head_dbl(const F& f, double x) 
	: f_(f), 
	  x_(x)
      { }

      // T2 can't just be T because remaining types change
      template <typename T2>
      stan::agrad::var
      operator()(const T2& t,
		 std::vector<stan::agrad::var>& xs) const {
	cons<double,T2> c(x_,t);
	return f_(c,xs);
      }

    };


    template <typename T>
    struct promote_cons {
    };
    template <>
    struct promote_cons<nil> {
      typedef double return_t;
    };
    template <typename H, typename T>
    struct promote_cons<cons<H,T> > {
      typedef 
      typename boost::math::tools::promote_args<H,
			   typename promote_cons<T>::return_t>::type
      return_t;
    };


    template <typename F>
    struct ignore_last_arg {
      const F& f_;
      ignore_last_arg(const F& f) : f_(f) { }

      template <typename C>
      typename promote_cons<C>::return_t
      operator()(const C& x) const {
	return f_(x);
      }

      template <typename C, typename T>
      typename promote_cons<C>::return_t
      operator()(const C& x, T& v) const {
	return f_(x);
      }
    };



}
}

```

Examples of a value test:

```

using std::vector;

using boost::math::tools::promote_args;

using stan::agrad::var;

using stan::test::bind_head_dbl;
using stan::test::bind_head_var;
using stan::test::cf;
using stan::test::cons;
using stan::test::ignore_last_arg;
using stan::test::nil;
using stan::test::promote_cons;




template <typename V, typename F, typename T>
void test_funct(const V& fx,
		const F& f,
		const T& x);



template <typename V, typename F>
void test_eq_rev(const V& fx,
		 const F& f,
		 const nil& x,
		 vector<double>& xs) {
  vector<var> xs_v(xs.size());
  for (size_t i = 0; i < xs.size(); ++i)
    xs_v[i] = xs[i];
  nil n;
  var fx_v = f(n,xs_v);
  EXPECT_FLOAT_EQ(fx, fx_v.val());
}
template <typename V, typename F, typename T>
void test_eq_rev(const V& fx,
		 const F& f,
		 const cons<double,T>& x,
		 vector<double>& xs) {
  bind_head_dbl<F,T> bh_rev(f,x.h_);
  test_eq_rev(fx, bh_rev, x.t_, xs);
  
  bind_head_var<F,T> bh_rev_var(f,xs.size());
  vector<double> xs_copy(xs);
  xs_copy.push_back(x.h_);
  test_eq_rev(fx, bh_rev_var, x.t_, xs_copy);
}

// test reverse-mode in all args
template <typename V, typename F, typename C>
void test_eq_rev(const V& fx,
		 const F& f,
		 const C& x) {
  vector<double> xs;
  test_eq_rev(fx,ignore_last_arg<F>(f),x,xs);
}


template <typename V, typename F, typename T>
void test_funct(const V& fx,
		const F& f,
		const T& x) {
  test_eq_rev(fx, f, x);
}

```

Example of how a developer would implement tests using the code above.

```


// ACTUAL TESTS START HERE
#include <typeinfo>

template <typename T1, typename T2>
typename boost::math::tools::promote_args<T1,T2>::type
multiply_base(const T1& x1, const T2& x2) {
  std::cout << "T1=" << typeid(T1).name()
	    << ";  T2=" << typeid(T2).name() << std::endl;
  return x1 * x2;
}

struct multiply_f {
  template <typename T1, typename T2>
  typename promote_args<T1,T2>::type
  operator()(const cons<T1, cons<T2,nil> >& x) const {
    return multiply_base(x.h_, x.t_.h_);
  }
};


TEST(Foo, Bar) {
  test_funct(6.0, multiply_f(), cf(3.0,cf(2.0)));
}
```

We have a few options for the interaction with the test framework. One option is to have the developer explicitly call each test, like so:

```

TEST(foo, bar1) {
  test_funct(6.0, multiply_f(), (3.0, 2.0));
  test_funct(-6.0, multiply_f(), (3.0, -2.0));
  test_funct(6.0, multiply_f(), (3.0, -2.0));
  test_funct(6.0, multiply_f(), (3.0, 2.0));
  test_funct(6.0, multiply_f(), (3.0, 2.0));
}

TEST(foo, invalid) {
  test_funct_invalid();
}

```

The other is to have a single call to the test framework but require subclass definitions, as in src/test/unit/prtest_fixture_distr.hpp. 

### Testing for "similarity"

In many tests, we compare values computed by two different mechanisms, without any assurance any one of them is "correct". For this case there is the `stan::test::expect_near_rel` function which handles NA and Inf values and uses the `stan::test::relative_tolerance` class to express tolerances.

The main idea about tolerances is that we care mainly about relative tolerance, but this fails around zero, so we use absolute tolerance around zero. I.e. the actual relative tolerance grows as we approach zero.

More details and reasoning behind this can be found in the code `relative_tolerance`, [on Dicourse](https://discourse.mc-stan.org/t/expect-near-rel-behaves-weirdly-for-values-around-1e-8/12573/6) and in [PR #1657](https://github.com/stan-dev/math/pull/1657).  
                                                               