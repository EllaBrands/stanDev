### Substance

#### Indexing Containers

For container indexes, use:

* `size_t` for `std::vector<S>`,

* `int` for `Eigen::Matrix<S, R, C>`, and

* `stan::math::index_type<T>::type` for template params `T` that might be instantiated as either a `std::vector<S>` or `Eigen::Matrix<S, R, C>`.

One could argue we should always use `Eigen::Matrix<S, R, C>::size_type` for Eigen constructs just in case we ever decide that we want to change their default indexing from `int` to something else.  But `int` should be OK for now because if we change the index type for some reason, the compiler will let us find instances.

### Style

We will follow the [Google Style Guide](https://google-styleguide.googlecode.com/svn/trunk/cppguide.html) except for the following exceptions.

##### Emacs mode for Google Style

http://google-styleguide.googlecode.com/svn/trunk/google-c-style.el


##### Named namespaces

* allow indentation inside namespaces
* allow skipping of `// namespace foo` after closing bracket of namespacen

##### Nonmember functions

* We'll have trouble with "Nonmember functions should not depend on external variables" due to our use of globals for memory management

##### Static and Global

* Not sure about how this impacts our memory management:

<blockquote>
Static or global variables of class type are forbidden: they cause hard-to-find bugs due to indeterminate order of construction and destruction. However, such variables are allowed if they are constexpr: they have no dynamic initialization or destruction.
</blockquote>

What they suggest as a workaround is this:

<blockquote>
If you need a static or global variable of a class type, consider initializing a pointer (which will never be freed), from either your main() function or from pthread_once().
</blockquote>

The issue with that is that it'll have to be done in the interfaces (PyStan, RStan, CmdStan).


* Because of our autodiff, we can't comply with:

<blockquote>
Objects with static storage duration, including global variables, static variables, static class member variables, and function static variables, must be Plain Old Data (POD): only ints, chars, floats, or pointers, or arrays/structs of POD.
</blockquote>

Otherwise, we should follow it. 

##### Exceptions

* We will allow exceptions.

##### Explicit Constructors

* This won't work for our autodiff type `var()` and `fvar()`, but otherwise sounds like a good idea:feasible:

<blockquote>
Use the C++ keyword explicit for constructors callable with one argument.
</blockquote>

I don't quite think our uses match their exception:

<blockquote> 
Classes that are intended to be transparent wrappers around other classes are also exceptions.
</blockquote>

##### Friends

* I'm not sure if our use of `var` will allow this:

<blockquote>
Friend declarations should always be in the private section.
</blockquote>


##### Reference Arguments

* I strongly disagree with this one, which they claim is a "very
strong convention in Google code"

<blockquote>
All parameters passed by reference must be labeled const.
</blockquote>

They want us to use pointers, the reason being:

<blockquote>
References can be confusing, as they have value syntax but pointer
semantics.
</blockquote>

So much for their injunction to assume readers of the code will know C++.

##### Run-time Type Information

* We have to violate this for the existing exception hierarchy in throwing exceptions with line numbers from Stan programs (otherwise I agree).

<blockqote>
Avoid using Run Time Type Information (RTTI).
</blockquote>

##### Streams

* No, no, no.  they want `printf` of all things (which causes runtime errors).

<blockquote>
Use streams only for logging.
</blockquote>


##### Inline Functions

* I think this is because they're inline by default, but I don't think it matters because the compiler's going to decide what to inline and there's also semantics for inline (can be redefined in multiple translation units):

<blockquote>
Do not put large method definitions inline in the class definition.
Usually, only trivial or performance-critical, and very short, methods
may be defined inline.
</blockquote>


##### Integer Types

* I think this is wrong because of `size_t`, and because of the crazy `__float128` extension:

<blockquote>
Of the built-in C++ integer types, the only one used is int.

If a program needs a variable of a different size, use a precise-width integer type from <stdint.h>, such as int16_t.
</blockquote>


##### Unsigned Integers

* Although I agree with the motivation to avoid subtle bugs, this messes up our other error checking that Daniel's been so careful to stomp (signed vs. unsigned comparisons):

<blockquote>
So, document that a variable is non-negative using assertions. Don't
use an unsigned type.
</blockquote>

##### Template Metaprogramming

* Too late for this (and they hate `enable_if`, but make an exception for general Boost packages like Boost Spirit):

<blockquote>
Avoid complicated template programming.
</blockquote>

##### Boost

* Of course, we use none of their approved modules (though Boost Spirit's not on it despite being mentioned elsewhere), but we use lots of other ones.

<blockquote>
Use only approved libraries from the Boost library collection.
</blockquote>


##### File names

* No, I like Boost and Eigen here, because they follow C++ rather
than C conventions:

<blockquote>
C++ files should end in .cc and header files should end in .h
</blockquote>

##### Type Names

* We went with Boost and Stroustroup, not Eigen and Google:

<blockquote>
Type names start with a capital letter and have a capital letter for each new word, with no underscores: MyExcitingClass, MyExcitingEnum.
</blockquote>

Though we haven't been entirely consistent (e.g., VectorView).

##### Variable Names

* We do this everywhere, including struct declarations.

<blockquote>
The names of variables and data members are all lowercase, with
underscores between words. Data members of classes (but not structs)
additionally have trailing underscores.
</blockquote>

##### Constant Names

* We went with the all-caps convention, not this:

<blockquote>
Use a k followed by mixed case, e.g., kDaysInAWeek, for constants defined globally or within a class.
</blockquote>

##### Function Names

* Again, we followed Stroustroup, not Google:

<blockquote>
Regular functions have mixed case; accessors and mutators match the name of the variable: MyExcitingFunction(), MyExcitingMethod(), my_exciting_member_variable(), set_my_exciting_member_variable().
</blockquote>

I think it's too late to change all of this as it'd be a massive
change.  And I really hate that they capitalize --- they should at
least follow Java's camel case, which would be "myExcitingFunction()"
to distinguish from types.

##### Comment Style

* I disagree with

<blockquote>
Use either the // or /* */ syntax, as long as you are consistent.
</blockquote>

We should always use // other than for doc comments.  The problem with /* ... */ is that you can't comment it out when you're debugging.  

##### File Comments

* I think we should avoid this:

<blockquote>
Start each file with license boilerplate, followed by a description of its contents.
</blockquote>

I just hate all the boilerplate.

They make author lines optional, and I think we should stay away from
them.  We have Git blame, after all.

* Just say no to this kind of redundancy:

<blockquote>
Every file should have a comment at the top describing its contents.
</blockquote>

Follow the authorial rule:  show, don't tell!



##### Pointer and Reference Expressions

* Let's stick to space following rather than preceding `&`, as in this example:

```
char* c;
const string& str;
``

##### Boolean Expressions

* They allow wraps either way, but I strongly prefer the typesetting convention of operators initial in lines, so prefer

```
if (this_one_thing > this_other_thing
    && a_third_thing == a_fourth_thing
    && yet_another && last_one) {
  ...
}
``

to

```
if (this_one_thing > this_other_thing &&
    a_third_thing == a_fourth_thing &&
    yet_another && last_one) {
  ...
}
```

##### Class Format

* This is unconventional for C++, but I'm OK with it, though don't like the one space. 

<blockquote>
Sections in public, protected and private order, each indented one space.

The basic format for a class declaration (lacking the comments, see Class Comments for a discussion of what comments are needed) is:

```
class MyClass : public OtherClass {
 public:      // Note the 1 space indent!
  MyClass();  // Regular 2 space indent.
  explicit MyClass(int var);
  ~MyClass() {}

  void SomeFunction();
  void SomeFunctionThatDoesNothing() {
  }

  void set_some_var(int var) { some_var_ = var; }
  int some_var() const { return some_var_; }

 private:
  bool SomeInternalFunction();

  int some_var_;
  int some_other_var_;
};
```
