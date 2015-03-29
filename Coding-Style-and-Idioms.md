# Substance

## Indexing Containers

For container indexes, use:

* `size_t` for `std::vector<S>`,

* `int` for `Eigen::Matrix<S, R, C>`, and

* `stan::math::index_type<T>::type` for template params `T` that might be instantiated as either a `std::vector<S>` or `Eigen::Matrix<S, R, C>`.

One could argue we should always use `Eigen::Matrix<S, R, C>::size_type` for Eigen constructs just in case we ever decide that we want to change their default indexing from `int` to something else.  But `int` should be OK for now because if we change the index type for some reason, the compiler will let us find instances.

# Style

We will follow the [Google Style Guide](https://google-styleguide.googlecode.com/svn/trunk/cppguide.html) except for the exceptions listed in the following sections.

`cpplint.py` is a Python (2.7) script that checks for these guidelines. Rhere is a make target that will check the source code for any instances that do not conform to this standard:

```
> make cpplint
```

## Emacs mode for Google Style

http://google-styleguide.googlecode.com/svn/trunk/google-c-style.el


## Exceptions to the Google Style Guide

Listed below are Stan's exceptions to the Google Style Guide. These rules that correspond to running cpplint with a filter argument are marked with `[category/rule]`.

These rules are ordered in the same order as the Google Style Guide.

### [Header Files](https://google-styleguide.googlecode.com/svn/trunk/cppguide.html#Header_Files)

#### [Self-contained Headers](https://google-styleguide.googlecode.com/svn/trunk/cppguide.html#Self_contained_Headers)

    Header files should be self-contained and end in .h. Files that 
    are meant for textual inclusion, but are not headers, should end 
    in .inc. Separate -inl.h headers are disallowed.
    
Our header files are `.hpp` files.
    

### [Scoping](https://google-styleguide.googlecode.com/svn/trunk/cppguide.html#Scoping)

#### [Namespaces](https://google-styleguide.googlecode.com/svn/trunk/cppguide.html#Namespaces) `[runtime/indentation_namespace, readability/namespace]`

**Named Namespaces**

* allow indentation inside namespaces 
* allow skipping of `// namespace foo` after closing bracket of namespace

#### [Nonmember, Static Member, and Global Functions](https://google-styleguide.googlecode.com/svn/trunk/cppguide.html#Nonmember,_Static_Member,_and_Global_Functions)

We'll have trouble with "Nonmember functions should not depend on external variables" due to our use of globals for memory management

#### [Static and Global Variables](https://google-styleguide.googlecode.com/svn/trunk/cppguide.html#Static_and_Global_Variables)

Not sure about how this impacts our memory management:

<blockquote>
Static or global variables of class type are forbidden: they cause hard-to-find bugs due to indeterminate order of construction and destruction. However, such variables are allowed if they are constexpr: they have no dynamic initialization or destruction.
</blockquote>

What they suggest as a workaround is this:

<blockquote>
If you need a static or global variable of a class type, consider initializing a pointer (which will never be freed), from either your main() function or from pthread_once().
</blockquote>

The issue with that is that it'll have to be done in the interfaces (PyStan, RStan, CmdStan).

Because of our autodiff, we can't comply with:

<blockquote>
Objects with static storage duration, including global variables, static variables, static class member variables, and function static variables, must be Plain Old Data (POD): only ints, chars, floats, or pointers, or arrays/structs of POD.
</blockquote>

Otherwise, we should follow it. 

### [Classes](https://google-styleguide.googlecode.com/svn/trunk/cppguide.html#Classes)

#### [Explicit Constructors](https://google-styleguide.googlecode.com/svn/trunk/cppguide.html#Explicit_Constructors)

This won't work for our autodiff type `var()` and `fvar()`, but otherwise sounds like a good idea:feasible:

<blockquote>
Use the C++ keyword explicit for constructors callable with one argument.
</blockquote>

I don't quite think our uses match their exception:

<blockquote> 
Classes that are intended to be transparent wrappers around other classes are also exceptions.
</blockquote>


#### [Declaration Order](http://google-styleguide.googlecode.com/svn/trunk/cppguide.html#Declaration_Order)

**Inline class definition**

I think this is because they're inline by default, but I don't think it matters because the compiler's going to decide what to inline and there's also semantics for inline (can be redefined in multiple translation units):

<blockquote>
Do not put large method definitions inline in the class definition.
Usually, only trivial or performance-critical, and very short, methods
may be defined inline.
</blockquote>


### [Other C++ Features](https://google-styleguide.googlecode.com/svn/trunk/cppguide.html#Other_C++_Features)

#### [Reference Arguments](https://google-styleguide.googlecode.com/svn/trunk/cppguide.html#Reference_Arguments) `[runtime/references]`

I strongly disagree with this one, which they claim is a "very
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

#### [Friends](https://google-styleguide.googlecode.com/svn/trunk/cppguide.html#Friends)

I'm not sure if our use of `var` will allow this:

<blockquote>
Friend declarations should always be in the private section.
</blockquote>

#### [Exceptions](https://google-styleguide.googlecode.com/svn/trunk/cppguide.html#Exceptions)

We will allow exceptions.

#### [Run-time Type Information (RTTI)](https://google-styleguide.googlecode.com/svn/trunk/cppguide.html#Run-Time_Type_Information__RTTI_)

We have to violate this for the existing exception hierarchy in throwing exceptions with line numbers from Stan programs (otherwise I agree).

<blockqote>
Avoid using Run Time Type Information (RTTI).
</blockquote>

#### [Streams](https://google-styleguide.googlecode.com/svn/trunk/cppguide.html#Streams)

 No, no, no.  they want `printf` of all things (which causes runtime errors).

<blockquote>
Use streams only for logging.
</blockquote>

#### [Integer Types](http://google-styleguide.googlecode.com/svn/trunk/cppguide.html#Integer_Types)

I think this is wrong because of `size_t`, and because of the crazy `__float128` extension:

<blockquote>
Of the built-in C++ integer types, the only one used is int.

If a program needs a variable of a different size, use a precise-width integer type from <stdint.h>, such as int16_t.
</blockquote>

**On Unsigned Integers**

Although I agree with the motivation to avoid subtle bugs, this messes up our other error checking that Daniel's been so careful to stomp (signed vs. unsigned comparisons):

<blockquote>
So, document that a variable is non-negative using assertions. Don't
use an unsigned type.
</blockquote>


#### [Template Metaprogramming](http://google-styleguide.googlecode.com/svn/trunk/cppguide.html#Template_metaprogramming)

Too late for this (and they hate `enable_if`, but make an exception for general Boost packages like Boost Spirit):

<blockquote>
Avoid complicated template programming.
</blockquote>

#### [Boost](http://google-styleguide.googlecode.com/svn/trunk/cppguide.html#Boost)

Of course, we use none of their approved modules (though Boost Spirit's not on it despite being mentioned elsewhere), but we use lots of other ones.

<blockquote>
Use only approved libraries from the Boost library collection.
</blockquote>

### [Naming](https://google-styleguide.googlecode.com/svn/trunk/cppguide.html#Naming)

#### [File names](https://google-styleguide.googlecode.com/svn/trunk/cppguide.html#File_Names)

No, I like Boost and Eigen here, because they follow C++ rather
than C conventions:

    C++ files should end in .cc and header files should end in .h

#### [Type Names](http://google-styleguide.googlecode.com/svn/trunk/cppguide.html#Type_Names)

We went with Boost and Stroustroup, not Eigen and Google:

<blockquote>
Type names start with a capital letter and have a capital letter for each new word, with no underscores: MyExcitingClass, MyExcitingEnum.
</blockquote>

Though we haven't been entirely consistent (e.g., VectorView).

#### [Variable Names](http://google-styleguide.googlecode.com/svn/trunk/cppguide.html#Variable_Names)

We do this everywhere, including struct declarations.

<blockquote>
The names of variables and data members are all lowercase, with
underscores between words. Data members of classes (but not structs)
additionally have trailing underscores.
</blockquote>

#### [Constant Names](http://google-styleguide.googlecode.com/svn/trunk/cppguide.html#Constant_Names)

We went with the all-caps convention, not this:

<blockquote>
Use a k followed by mixed case, e.g., kDaysInAWeek, for constants defined globally or within a class.
</blockquote>

#### [Function Names](http://google-styleguide.googlecode.com/svn/trunk/cppguide.html#Function_Names)

Again, we followed Stroustroup, not Google:

<blockquote>
Regular functions have mixed case; accessors and mutators match the name of the variable: MyExcitingFunction(), MyExcitingMethod(), my_exciting_member_variable(), set_my_exciting_member_variable().
</blockquote>

I think it's too late to change all of this as it'd be a massive
change.  And I really hate that they capitalize --- they should at
least follow Java's camel case, which would be "myExcitingFunction()"
to distinguish from types.


### [Comments](http://google-styleguide.googlecode.com/svn/trunk/cppguide.html#Comments)

#### [Comment Style](http://google-styleguide.googlecode.com/svn/trunk/cppguide.html#Comment_Style)

I disagree with

<blockquote>
Use either the // or /* */ syntax, as long as you are consistent.
</blockquote>

We should always use // other than for doc comments.  The problem with /* ... */ is that you can't comment it out when you're debugging.  

#### [File Comments](http://google-styleguide.googlecode.com/svn/trunk/cppguide.html#File_Comments) `[legal/copyright]`

I think we should avoid this:

<blockquote>
Start each file with license boilerplate, followed by a description of its contents.
</blockquote>

I just hate all the boilerplate.

They make author lines optional, and I think we should stay away from
them.  We have Git blame, after all.

Just say no to this kind of redundancy:

<blockquote>
Every file should have a comment at the top describing its contents.
</blockquote>

Follow the authorial rule:  show, don't tell!


### [Formatting](http://google-styleguide.googlecode.com/svn/trunk/cppguide.html#Formatting)


#### [Pointer and Reference Expressions](http://google-styleguide.googlecode.com/svn/trunk/cppguide.html#Pointer_and_Reference_Expressions)

Let's stick to space following rather than preceding `&`, as in this example:

```
char* c;
const string& str;
```

#### [Boolean Expressions](http://google-styleguide.googlecode.com/svn/trunk/cppguide.html#Boolean_Expressions)

They allow wraps either way, but I strongly prefer the typesetting convention of operators initial in lines, so prefer

```
if (this_one_thing > this_other_thing
    && a_third_thing == a_fourth_thing
    && yet_another && last_one) {
  ...
}
```

to

```
if (this_one_thing > this_other_thing &&
    a_third_thing == a_fourth_thing &&
    yet_another && last_one) {
  ...
}
```

#### [Class Format](http://google-styleguide.googlecode.com/svn/trunk/cppguide.html#Class_Format) `[whitespace/indent]`

This is unconventional for C++, but I'm OK with it, though don't like the one space. **DL: I don't like one space.**

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




