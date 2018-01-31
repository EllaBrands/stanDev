We generally follow the [Google Style Guide](https://google.github.io/styleguide/cppguide.html) except for the exceptions listed in the following sections. We use a few tools to automate style and formatting checking, but they don't completely capture our formatting and style guidelines.

## Automated style checking tools

### cpplint
`cpplint.py` is a Python (2.7) script that checks for these guidelines. Here is a make target that will check the source code for any instances that do not conform to this standard:

```
> make cpplint
```

If your default python version is python 3, the cpplint.py script fails silently. Once you install python 2.x, you can add this to your `make/local` file to use python 2.x to run the python script:
```
RUN_CPPLINT = python2 stan/lib/cpplint_4.45/cpplint.py
```

The make target includes Stan-specific options and limits the tests to the src/stan directory.

### clang-format
clang-format is a tool that will automatically format C++ code according to formatting rules in the `.clang-format` file. Our file is just a few changes from the vanilla Google Style guide. `clang-format` doesn't deal with many of our exceptions to the Style guide, below, as they are difficult to automate or not strictly formatting related.

#### Setup
First, [install `clang-format`](http://geant.cern.ch/content/clang-format-git-hook). If you're on a mac and using homebrew, please install clang-format@2017-11-14 with this command:
```
brew install clang-format@2017-11-14
```
If you're already on a previous version of clang-format, you can upgrade with 
```
brew upgrade clang-format@2017-11-14
```
(if that doesn't work, try `brew install clang-format` or `brew upgrade clang-format` and get whatever your computer thinks is the latest)
If you're on Ubuntu Linux, you can use if you're on Ubuntu you can use the LLVM PPA to install clang-format-5.0 and then use update-alternatives to set it as the default. Find the relevant 5.0 PPA for your version of Ubuntu here: https://apt.llvm.org/ and follow their PPA install instructions, then:
```
sudo apt-get install clang-format-5.0
sudo update-alternatives --install /usr/bin/clang-format clang-format /usr/bin/clang-format-5.0 100
```

##### Editor setup
Emacs - there's a plugin/script called `google-c-style`. You can find it [here](https://raw.githubusercontent.com/google/styleguide/gh-pages/google-c-style.el) or just install from MELPA.

Spacemacs - you can add `google-c-style` to `dotspacemacs-additional-packages`.

Vim- check out https://github.com/google/vim-codefmt

##### Git hook
There is a pre-commit hook in hooks/pre-commit, and if you run `bash hooks/install_hooks.sh` it will be installed for you. Going forward, it will run `clang-format -i` (which implicitly uses the `.clang-format` file in the repo) on any changed files to format them. If you need to disable this for some reason (though your tests will fail if you haven't formatted things correctly) you can use `git commit --no-verify`.

##### Differences from clang-format's `--style=google`
To see which rules are different, you can run this command:
```sh
clang-format --style=google -dump-config > google
diff google .clang-format 
```
And here's the current output:
```
15,16c15,16
< AllowShortIfStatementsOnASingleLine: true
< AllowShortLoopsOnASingleLine: true
---
> AllowShortIfStatementsOnASingleLine: false
> AllowShortLoopsOnASingleLine: false
36c36
< BreakBeforeBinaryOperators: None
---
> BreakBeforeBinaryOperators: All
86c86
< SortIncludes:    true
---
> SortIncludes:    false
```

You can see the rule definitions [here.](https://clang.llvm.org/docs/ClangFormatStyleOptions.html) 

## Exceptions to the Google Style Guide

Listed below are Stan's exceptions to the Google Style Guide. These rules that correspond to running cpplint with a filter argument are marked with `[category/rule]`.

These rules are ordered in the same order as the Google Style Guide.

### [Header Files](https://google.github.io/styleguide/cppguide.html#Header_Files)

#### [Self-contained Headers](https://google.github.io/styleguide/cppguide.html#Self_contained_Headers)

    Header files should be self-contained and end in .h. Files that 
    are meant for textual inclusion, but are not headers, should end 
    in .inc. Separate -inl.h headers are disallowed.
    
Our header files are `.hpp` files.
   
#### [Nonmember, Static Member, and Global Functions](https://google.github.io/styleguide/cppguide.html#Nonmember,_Static_Member,_and_Global_Functions)

We'll have trouble with "Nonmember functions should not depend on external variables" due to our use of globals for memory management

#### [Static and Global Variables](https://google.github.io/styleguide/cppguide.html#Static_and_Global_Variables)

Not sure about how this impacts our memory management:

> Static or global variables of class type are forbidden: they cause hard-to-find bugs due to indeterminate order of construction and destruction. However, such variables are allowed if they are constexpr: they have no dynamic initialization or destruction.

What they suggest as a workaround is this:

> If you need a static or global variable of a class type, consider initializing a pointer (which will never be freed), from either your main() function or from pthread_once().

The issue with that is that it'll have to be done in the interfaces (PyStan, RStan, CmdStan).

Because of our autodiff, we can't comply with:

> Objects with static storage duration, including global variables, static variables, static class member variables, and function static variables, must be Plain Old Data (POD): only ints, chars, floats, or pointers, or arrays/structs of POD.

Otherwise, we should follow it. 

### [Classes](https://google.github.io/styleguide/cppguide.html#Classes)

#### [Explicit Constructors](https://google.github.io/styleguide/cppguide.html#Explicit_Constructors)

This won't work for our autodiff type `var()` and `fvar()`, but otherwise sounds like a good idea:feasible:

> Use the C++ keyword explicit for constructors callable with one argument.

I don't quite think our uses match their exception:

> Classes that are intended to be transparent wrappers around other classes are also exceptions.

#### [Declaration Order](https://google.github.io/styleguide/cppguide.html#Declaration_Order)

**Inline class definition**

I think this is because they're inline by default, but I don't think it matters because the compiler's going to decide what to inline and there's also semantics for inline (can be redefined in multiple translation units):

> Do not put large method definitions inline in the class definition.
> Usually, only trivial or performance-critical, and very short, methods
> may be defined inline.


### [Other C++ Features](https://google.github.io/styleguide/cppguide.html#Other_C++_Features)

#### [Reference Arguments](https://google.github.io/styleguide/cppguide.html#Reference_Arguments) `[runtime/references]`

I strongly disagree with this one, which they claim is a "very
strong convention in Google code"

> All parameters passed by reference must be labeled const.

They want us to use pointers, the reason being:

> References can be confusing, as they have value syntax but pointer
semantics.

So much for their injunction to assume readers of the code will know C++.

#### [Friends](https://google.github.io/styleguide/cppguide.html#Friends)

I'm not sure if our use of `var` will allow this:

> Friend declarations should always be in the private section.

#### [Exceptions](https://google.github.io/styleguide/cppguide.html#Exceptions)

We will allow exceptions.

#### [Run-time Type Information (RTTI)](https://google.github.io/styleguide/cppguide.html#Run-Time_Type_Information__RTTI_)

We have to violate this for the existing exception hierarchy in throwing exceptions with line numbers from Stan programs (otherwise I agree).

> Avoid using Run Time Type Information (RTTI).

#### [Streams](https://google.github.io/styleguide/cppguide.html#Streams)

 No, no, no.  they want `printf` of all things (which causes runtime errors).

> Use streams only for logging.

#### [Integer Types](https://google.github.io/styleguide/cppguide.html#Integer_Types)

I think this is wrong because of `size_t`, and because of the crazy `__float128` extension:

 > Of the built-in C++ integer types, the only one used is int.

> If a program needs a variable of a different size, use a precise-width integer type from <stdint.h>, such as int16_t.

**On Unsigned Integers**

Although I agree with the motivation to avoid subtle bugs, this messes up our other error checking that Daniel's been so careful to stomp (signed vs. unsigned comparisons):

> So, document that a variable is non-negative using assertions. Don't
use an unsigned type.


#### [Template Metaprogramming](https://google.github.io/styleguide/cppguide.html#Template_metaprogramming)

Too late for this (and they hate `enable_if`, but make an exception for general Boost packages like Boost Spirit):

> Avoid complicated template programming.

#### [Boost](https://google.github.io/styleguide/cppguide.html#Boost)

Of course, we use none of their approved modules (though Boost Spirit's not on it despite being mentioned elsewhere), but we use lots of other ones.

> Use only approved libraries from the Boost library collection.

### [Naming](https://google.github.io/styleguide/cppguide.html#Naming)

#### [File names](https://google.github.io/styleguide/cppguide.html#File_Names)

No, I like Boost and Eigen here, because they follow C++ rather
than C conventions:

    C++ files should end in .cc and header files should end in .h

#### [Type Names](https://google.github.io/styleguide/cppguide.html#Type_Names)

We went with Boost and Stroustroup, not Eigen and Google:

> Type names start with a capital letter and have a capital letter for each new word, with no underscores: MyExcitingClass, MyExcitingEnum.

Though we haven't been entirely consistent (e.g., VectorView).

#### [Variable Names](https://google.github.io/styleguide/cppguide.html#Variable_Names)

We do this everywhere, including struct declarations.

> The names of variables and data members are all lowercase, with
underscores between words. Data members of classes (but not structs)
additionally have trailing underscores.

#### [Constant Names](https://google.github.io/styleguide/cppguide.html#Constant_Names)

We went with the all-caps convention, not this:

> Use a k followed by mixed case, e.g., kDaysInAWeek, for constants defined globally or within a class.

#### [Function Names](https://google.github.io/styleguide/cppguide.html#Function_Names)

Again, we followed Stroustroup, not Google:

> Regular functions have mixed case; accessors and mutators match the name of the variable: MyExcitingFunction(), MyExcitingMethod(), my_exciting_member_variable(), set_my_exciting_member_variable().

I think it's too late to change all of this as it'd be a massive
change.  And I really hate that they capitalize --- they should at
least follow Java's camel case, which would be "myExcitingFunction()"
to distinguish from types.

### [Comments](https://google.github.io/styleguide/cppguide.html#Comments)

#### [Comment Style](https://google.github.io/styleguide/cppguide.html#Comment_Style)

I disagree with

> Use either the // or /* */ syntax, as long as you are consistent.

We should always use // other than for doc comments.  The problem with /* ... */ is that you can't comment it out when you're debugging.  

#### [File Comments](https://google.github.io/styleguide/cppguide.html#File_Comments) `[legal/copyright]`

I think we should avoid this:

> Start each file with license boilerplate, followed by a description of its contents.

I just hate all the boilerplate.

They make author lines optional, and I think we should stay away from
them.  We have Git blame, after all.

Just say no to this kind of redundancy:

> Every file should have a comment at the top describing its contents.

Follow the authorial rule:  show, don't tell!