## Testing framework

#### Current state of non-probability function math unit tests
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

Bob's first cut at a recursive template metaprogramming approach to automatically expanding the type combinations:

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