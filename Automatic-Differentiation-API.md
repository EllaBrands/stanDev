Stan's forward- and reverse-mode automatic differentiation API has been written so that it can be used by itself outside of the context of Stan's programming language and built-in inference systems. 

#### Distribution

There is currently no standalone distribution for just the autodiff, but you can just download all the Stan source and use it that way (easy) or do the same and remove all the unused namespaces (harder).  In the future we might create a standalone distribution for autodiff, but it's not high on our priority list.

#### High-Level Functionals

The easiest point of entry into Stan's autodiff is the collection of functionals found in `src/stan/agrad/autodiff.hpp`.  These can be used to automatically differentiate arbitrary templated functors in Stan without knowing anything at all about Stan's built-in automatic differentiation types.

Looking at the code implementing these functionals is also a good how-to on how to use the lower-level forward- and reverse-mode data types.

#### Unit Tests

The other place to look for examples is in our unit tests, which are currently (but not perhaps future-ly) in three places:

<ul>
<li> `src/test/unit/agrad`
<li> `src/test/unit-agrad-fwd`
<li> `src/test/unit-agrad-rev`
</ul>

There are only a handful of tests in the first directory and all these directories will probably get rearranged in the future.  

#### Reverse Mode


#### Forward Mode


#### Nesting Reverse in Forward Modes

 