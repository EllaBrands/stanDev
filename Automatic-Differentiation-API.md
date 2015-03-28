# THIS IS NOW OUT-OF-DATE WITH RESPECT TO THE develop BRANCH

## PLEASE CHECK BACK LATER --- WE HOPE TO GET IT UPDATED.

Stan's forward- and reverse-mode automatic differentiation API has been written so that it can be used by itself outside of the context of Stan's programming language and built-in inference systems. 

### Source Distribution

There is currently no standalone distribution for just the autodiff, but you can just download all the Stan source and use it that way (easy) or do the same and remove all the unused namespaces (harder).  In the future we might create a standalone distribution for autodiff, but it's not high on our priority list.

### High-Level Library Calls

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

### Reverse Mode

##### Recovering memory

In the basic case, there's nothing to do in the basic setup other than call `agrad::recover_memory()` after using reverse mode and saving everything necessary from the results.

Stan also allows nested evaluations of reverse mode, whereby a current position is saved, autodiff is done on top of the variable stack, then the stack is recovered back to the last saved position.  This is handled by first calling `agrad::start_nested()`, carrying out the nested autodiff as per usual reverse-mode autodiff, then calling `agrad::recover_memory_nested()` to recover the memory.

#### Forward Mode


#### Nesting Reverse in Forward Modes

See the examples in `autodiff.hpp`

 