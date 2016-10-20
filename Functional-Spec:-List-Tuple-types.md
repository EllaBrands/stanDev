I'd like to add a heterogeneous typed container to Stan's language.

## Declaring Tuples

A declaration would look something like this:

```
tuple<T1, ..., TN> x;
```

where `T1` through `TN` are sized type specifications.   Or we could use `seq` or `list` in place of `tuple` --- they're both shorter.

<b>Issue:</b>  This is a bit tricky in that we don't have a generic sized type system yet, only the unsized one used for functions and the one involved with declaring variables.  The obvious choices work for basic types, like `real` or `matrix[3, 2]`, but it's trickier for arrays.  For arrays of reals, that's also easy, because we can use something like `real[3, 2]` for a 2D array of reals.  For a 2D array of vectors, we'll have to use something like `vector[3][4, 5]`.  At that point, I thik I'd want to introduce these into the language itself as an alternative to the mixed form 

```
vector[3] x[4, 5];
```

where the size of the array comes after the variable, but the size of the basic type goes on the basic type.

## Heads, Tails, and Concatenations

It shold be possible to get the head or tail of a tuple.  

```
tuple<T2, ..., TN> tail(tuple<T1, T2, ..., TN> a);
T1 head(tuple<T1, ..., TN> a);
```

and concatenate them back into a tuple

```
tuple<T1, T2, ..., TN> cat(T1 a, tuple<T2, ..., TN> b);
```

## Element Indexing lvalues and rvalues

We should also be able to get (rvalue) and set (lvalue) by element

```
tuple<real, int, vector[2]> a;
real c;
int d;
vector[2] e;

c = a[1];
d = a[2];
e = a[3];

a[1] = c;
a[2] = d;
a[3] = e;
```

I don't think we should make `head()` and `tail()` return lvalues, though we could.

We probably also want an appending function that isn't overloaded with cat:

```
tuple<T1, ..., TN, U1, ..., UM> append(tuple<T1, ..., TN> a, tuple<U1, ..., UM> b);
```

## C++ Implementation

This is easy on the data-type side in C++:

```
struct nil_tuple {
  nil_tuple() { }
};

template <typename H, typename T>
class tuple {
private:
  H head_;
  T tail_;
public:
  tuple(const H& head, const T& tail) : head_(head), tail_(tail) { }
  H& head() { return head_; }  // const return?
  T& tail() { return tail_; }  // const return?
};
```

It's not possible to declare an `operator[]` member function because there's no way to calculate the return type statically.  Consider:

```
  ??? operator[](int n) { return n == 0 ? head() : tail()[n - 1]; }
```

The argument `n` is only known at run time and will determine return value of type.  We could always go with the lisp convention:

```
car(x) = head(x)
cdr(x) = tail(x)
cadr(x) = head(tail(x))
caddr(x) = head(tail(tail(x)))
...
```

We could try to get around this by providing a `void *` return type for `operator[]` and then having the Stan program cast it back to what it needs to be.


The only problem is that 'm having a bit of trouble seeing how to declare a return type for the indexing operator as the index won't be known until run time.
