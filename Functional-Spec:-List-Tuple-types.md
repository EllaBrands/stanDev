I'd like to add a heterogeneous typed container to Stan's language.

## Declaring Tuples

A declaration would look something like this:

```
(T1, ..., TN) x;
```

where `T1` through `TN` are sized type specifications.

<b>Issue:</b>  To make this fly, we need to have a way of declaring sized types contiguously.  Right now, declarations like `int x[3]` split the `int` and `[3]`.  Mitzi's working on this as part of the general refactor of the underlying type system.

## Tuple Expressions

The expression for creating tuples can follow the array notation (`{ a, b, c }`) and vector notation (`[a, b, c]`) and use `(a, b, c)`.  This will create a tuple of type `(typeof(a), typeof(b), typeof(c))`, which will be inferrable.  

## Tuple Accessors

We can't have general access by integer because we can't figure out the type statically and Stan is strongly statically typed.  So we'll have to hardcode the accessors, as in:

```
tuple(S, T, U) x;
...
S a = x.1;
T b = x.2;
U c = x.3;
// x.4 causes compilation error
```

And just to say this again, we can't have `x[n]` for some runtime-determined `n` because we need to determine return type statically.


## Heads, Tails, and Concatenations

We can follow Lisp and define some basic operations on tuples as lists, including heads (car),

```
T1 
head((T1, ..., TN) a);
```

tails (cdr),

```
(T2, ..., TN)
tail((T1, T2, ..., TN) a);
```

and concatenation (cat)

```
(T1, T2, ..., TN)
concat(T1 a, (T2, ..., TN) b);
```



## Lvalues

We want to allow tuples and elements of tuples on the left-hand side of assignments, as in:

```
(double, int) a;
a = (3.7, 2);  // assign to 
a.1 = 4.2;
```

## Promotion (Covariance)

Should tuples be fully covariant?  That means that whenever `a` is a subtype of `b` (e.g., `int` is a subtype of `real`), then `(..., a, ...)` is a subtype of `(..., b, ...)` in a tuple.  This would allow us to assign an integer and double tuple to a pair of double value tuples, `(int, double, matrix)` assignable to `(double, double, matrix)`.  If that doesn't work, the expression construction syntax is going to be confusing when `int` doesn't act like `double`.

## Alternative syntax

Instead of `(T1, ..., TN)`, we could use `tuple(T1, ..., TN)`.  It's a little more explicit.  The implicit version matches behavior in Python.  

## Not R lists

Tuple are not meant to be dynamic heterogeneous containers like lists in R.  The types are declared statically like every other variable in Stan.



## C++ Implementation

We can drop down to C++11 tuple types if they scale or use a Lisp-type encoding.

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
