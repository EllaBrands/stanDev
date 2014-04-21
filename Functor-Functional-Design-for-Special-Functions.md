#### General-Purpose Functional

There's a general-purpose functional that does the vectorization, which is here a simple elementwise application I called `distribute`.  First, a recursive case for vectors:

```
  template <typename F, typename T>
  vector<T> distribute(const vector<T>& x) {
    vector<T> y(x.size());
    for (size_t i = 0; i < x.size(); ++i)
      y[i] = distribute<F>(x[i]);
    return y;	 
  }
```

and then a base case for scalars:

```
  template <typename F, typename T>
  T distribute(const T& x) {
    return F::apply(x);
  }
```

#### Functor for Calculation

There's a developer-defined functor to actually do the calculation for a given special function:

```
  class Phi_functor {
    static double apply(double x) {
      ...
    }
  }
```

#### Function Definition

The special function itself is defined by applying the functional to the functor:

```
  template <typename T>
  T
  Phi(const T& x) {
    return distribute<Phi_functor>(x);
  }
```

#### Auto-diff

If we extend the `Phi_functor` to also include derivative and trait (e.g., whether
there are non-zero 2nd order partials) information, then we can also key auto-diff
off of similar apply functionals.