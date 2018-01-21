Stan requires care and attention to symbol resoluition in the autodiff types.  

We use the namespace `stan::math` (in the sense of having a top-level using statement for it all), so all the symbols from that namespace are available.  

`std::log1p(double)` is part of the standard template library.  `std::log1p(int)` is not.  It relies on automatic promotion of `int` to `double`.  

`stan::math::log1p(stan::math::var)` is part of Stan---it's what we use for automatic differentiation types.

Now the problem arises that if we see `log1p(int)`, it's ambiguous.  The `int` can be promoted to double to match `std::log1p(double)`, or the int can be promoted to stan::math::var (because there's an implicit constructor `stan::math::var(int))`.  

To disambiguate, we define `stan::math::log1p(int)`, which is more specific than either `std::log1p(double)` or `stan::math::log1p(stan::math::var)`.

Now the question is whether we want the exception behavior of `log1p` to match the standard library or match the rest of the Stan library.  

If we want it to match the rest of the Stan library, we need to define `stan::math::log1p(double)` with the appropriate behavior.  Then we need to make sure that `log1p(double)` calls `stan::math::log1p` rather than `std::log1p`.

That then introduces the headache that if we are in a scope that's using `std::log1p(double)`, we get an ambiguity with `stan::math::log1p(double)`.  

It's further complicated by the existence of `::log1p(double)` in the top-level namespace in the old `.h` forms of the C headers for C++.  It's further complicated still  by the fact that C++ decided to leave the size (and hence ambiguity) of various declarations (like int and long) up to the compiler writer.

