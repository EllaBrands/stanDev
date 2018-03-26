The basic idea of parallelism using the message passage interface is that we can use autodiff just as is, since MPI will start *independent* processes. All what MPI does is to organize the exchange of messages between the different processes.

# Requirements

Guess what...a working base MPI installation is needed. The base MPI installation provides the command line tools
+ `mpicxx`: The recommended  compiler command to use when building any MPI application.
+ `mpirun`: Wrapper binary used to start a MPI enabled binary on a given machine.

Currently Stan supports MPI on macOS and Linux. For these platforms all MPI base installations are supported which are supported by `boost.mpi`. However, Stan testing is only performed with the `mpich` base system. For macOS macports or homebrew have a `mpich` package available. For Linux `mpich` (or `openmpi`) should be available through the package manager of your distribution.

Note that Stan builds it's own `boost.mpi` & `boost.serialization` libraries and installs these into it's lib sub-folder. Should the OS provide these boost libraries and there is a need to use them, then the build system needs to be adjusted for this (through `make/local`). By default both libraries are build using `boost.build` which should recognize the base MPI system available.

# Building `stan-math` with MPI

A working base MPI installation (`mpirun` & `mpicxx`) is required for Stan's MPI facilities. Stan uses the `boost.mpi` library to interface with the low-level C-library MPI base system. The `boost.mpi` furthermore depends on the `boost.serialization` library which is build automatically once needed. To activate MPI for Stan please proceed as follows:

0. Ensure that a base MPI installation is available and accessible on the system. Stan's tests are executed using the `mpich` distribution, but any other base MPI installation compatible with `boost.mpi` should work. See the [instructions from `boost.mpi`](http://www.boost.org/doc/libs/1_66_0/doc/html/mpi/getting_started.html#mpi.mpi_impl) for help on ensuring that the base MPI system works.
1. Put `STAN_MPI=true` into the `make/local` file.
2. Put `CC=mpicxx` into the `make/local` file. Alternatively, the user may specify the compiler to use with all compiler and linker options needed to build a MPI enabled binary (the command `mpicxx -show` displays for `mpich` what is executed / `openmpi` uses `mpicxx -show-me`).

Once `STAN_MPI` and the compiler is set accordingly, then Stan will automatically build MPI enabled binaries. Note that the `boost.mpi` and `boost.serialization` library are build and linked against dynamically. In order to make it easy to use these libraries the paths for the dynamic libraries are hard-coded into the generated binaries. The `-rpath` option of the linker is used to achieve this. While this works out of the box on Linux, a few additional steps are needed on macOS during library building which are all handled in full automation by the makfiles.

# How to run MPI tests in `stan-math`

All units tests in `stan-math` are run in MPI mode with the `mpich` base installation on the Jenkins machine. There are two types of tests to discriminate:

- conventional tests: The comprise all unit tests as we run them in serial non-MPI mode. In order to run these tests in a MPI like way we compile these with the `mpicxx` command and execute them with the `mpirun` run command. However, we explicitly disable for these serial tests the use of multiple processes. That is, the `runTests.py` script executes the tests with `mpirun -np 1 test/unit/.../.../some_test`. Starting up multiple threads for serial only tests would lead to race conditions since these codes are not prepared for parallelism.

- dedicated mpi tests: All tests matching the regular expression `*mpi_*test.cpp` will be executed by `runTests.py` with `mpirun -np #CPU test/unit/.../.../some_mpi_test.cpp` and #CPU will be set to the same argument as given to the `-j` option of `runTests.py`, but it will use are least 2 processes. This is to ensure that the MPI tests are actually run with multiple processes in parallel to emulate behavior under MPI execution. Note that `mpirun` is usually configured to disallow #CPU to exceed the number of physical CPUs found on the machine. 

# The rest of this wiki is mostly outdated...

... so ignore for now.

# Goals

- within-chain parallelization
- general construct working with any Stan program
- efficiency / minimized communication
- possibility of different backends such that non-MPI environments can run the code

# Non-Goals

- super user-friendlyness not required, index war is OK
- dynamic allocation of jobs; would be nice to have, but this can be left for a later implementation
- (?) multiple regions of parallelism (?)

# Future Goals

- Sparsity detection. For higher order autodiff many derivates will be zero (right?) in many cases. Therefore, a sparse communication should speed things up for such applications.

# Open points, discussed Sept 4th

*italic text summarizes decision* during meeting from Sept 4th with Sebastian, Bob, Charles, Daniel and Sean. **bold items** mark action items:

- Finalize design (maybe I pair program with Bob?)
   - Naming and placement of MPI utilities (new mpi namespace? use of an internal namespace?)
   - A minimal unit test is here: https://github.com/stan-dev/math/blob/feature/concept-mpi/test/unit/math/rev/arr/functor/map_rect_mpi_test.cpp
   - And here is how this integrates with cmdstan: https://github.com/stan-dev/cmdstan/blob/feature/proto-mpi/src/cmdstan/main-mpi2.cpp

*most MPI stuff will be crammed into an internal namespace. After another clean sweep from Sean and eventual iteration with Sebastian we will kick off a pull-request.* **Sean to review code in stan-math/feature/concept-mpi and align with Sebastian on design**.

- Building: Building against a shared boost mpi and boost serialization works, but static linking looks tricky => do we need to ensure a static build process?

*static building preferred, but not a must. We will target Mac and Linux first; Windows is optional.* **Daniel to look into details as to how to integrate and if static build can be done.**

- Testing: Additional *external* dependencies! How to manage this? Travis?
   - Additional build dependencies
   - MPI OS dependency
   - Stan test program must be called with `mpirun`

*We will test MPI stuff with extra special code on internal Jenkins. No Travis.*

- How to handle nested map calls? Disallow them?

*We will disallow them most likely in one or the other way (runtime or better at Stan compile time). I just implemented a runtime lock based thing such that there can never be two dispatched commands at a time.*

- The current proposal caches the data after its first send.
   - The user gives a uid for dataset identification. OK?
   - Do we need a flush operation which clears the cache?
   - Can we assume a smart user who will always use the same uid for the same data set or do we need some checks (which can only be ad-hoc unless we pay the price for hashing)?

*Burdening C++ users to manage uids identifying immutable data is OK. No flush operation for now. C++ users are smart by definition.*

- Start of more user-friendly version as agreed here: http://discourse.mc-stan.org/t/mpi-design-discussion/1103/38?u=wds15

**Bob to start stan language work. Bob, Daniel and Charles offered to write unit tests once pull-requests starts.**

Next steps:

- Sean and Sebastian align on design (naming, placement, ...)
- Open pull-request
- Unit tests / build system
- Stan language

We intend to start with the simple `map_rect` version first which does not include a shared parameter vector. What's next we will decide along (options are shared/not shared and aggregating/not aggregating).

Sebastian answered, 2nd Sept:
- Name for the new function? `map_rect`
- Do we want to enable multiple `map` calls in a Stan program or just allow a single one? Yes, we want to allow multiple ones.
- Is the ragged array construct what we want to use (it really is a workaround)? No, we can go with a single version.
- How is the backend configured? At compile time? Yes, at compile time.
- Backends we need are: MPI, serial? That's all.

# Efficiency

To warrant good efficiency we need to

- minimize the communication
- try to ship static data only once
- only use `boost::mpi` constructs which avoid the `boost::serialization`, but can use the C implementations of the MPI implementation which are much more efficient and faster.

# Stan language

The current proposal is to define a parallelizable `map()` function of the form found in languages such as Lisp.

```
/**
 * @param f  the function to map, with signature `(real[] params, real[] x_r, int[] x_i) : real[]`
 * @param y_end  ragged end points for the returned value
 * @param param  ragged parameter values
 * @param param_end  ragged parameter vector end points
 * @param x_r  ragged real data values
 * @param x_r_end  ragged real data end points
 * @param x_i  ragged integer data values
 * @param x_i_end  ragged integer data end points
 * @return ragged return values
 */
real[] map(F f, int[] y_end,
           real[] param, int[] param_end,
           real[] x_r, int[] x_r_end,
           int[] x_i, int[] x_i_end) {

  ... validate all end sequences are 
      same size, ascending order, non-negtive ...

  int N = size(y_end);  // number of function calls
  real[y_end[N]] y;     // return value
  int y_start = 1;      // ragged array start points
  int param_start = 1;
  int x_r_start_ = 1;
  int x_i_start = 1;

  // IN PARALLEL:
  for (i in 1:N) {
    y[y_start:y_end[i]] = f(param_start[param_start:param_end[i]], 
                            x_r[x_r_start:x_r_end[i]],
                            x_i[x_i_end:x_i_end[i]]);
    param_start = param_end[i] + 1;
    x_r_start = x_r_end[i] + 1;
    x_i_start = x_i_end[i] + 1;
    y_start = y_end[i] + 1;
  }

  return y;
}
```

The *in parallel* section will proceed in the steps

1. ship all parameters block-wise to the worker nodes. I.e. the blocking is fixed by the order in advance. This ensures that the communication is done in a single go at once which can be implemented with the MPI `scatter` command in MPI.

2. Each worker performs their packet of work. This includes function evaluation and gradient computation.

3. All results are collected using the MPI `gather` command. This is done such that the results are directly written to the AD memory stack of the root node.

4. On the root node, the results are injected into the AD tree with the `precomputed_gradients` approach using pointers to avoid unnecessary copies.

The above procedure assumed that the data has been shipped once to the workers. This is doable given that we can detect that the data is in scope of the `map` construct.   *[Question:  is there a way to do this on the first call and then set a flag we can check later to make sure the variables have already been shipped?]*

We'll also need a serial implementation of `map()` which pretty much just implements the pseudocode.  

# Notes on the prototype

The `cmdstan` implementation works roughly as follows:

1. A modified `main.cpp` is used. The MPI environment is initialized in the `main-mpi.cpp` and the `env.abort()` is called on the main process to shutdown all workers when the root terminates.  *[Question:  Would it be easier to generate a single `main.cpp` and have the compilation of it controlled by some kind of flag?]*

2. The root and the workers start normally. All processes call at the end of the `transformed data` block a `setup_mpi_function`.

3. The `setup_mpi_function` determines the `y_out` (ragged structure of the function outputs) and behaves differently on  the different nodes
   - workers go into an infinite loop where they wait for packets of parameters from the root; in addition the workers subset the data to the set of data which is allocated the the worker *[Question:  Can we do this more efficiently than spin-waiting?]* 
   - the root: here the function returns with a data structure which maps each job to a worker id and the output sizes per job

4. only on the root node the `run_mpi_function` is ever called in the `transformed parameters` block which does
   - sent blockwise the parameters to the workers using the MPI scatter command
   - allocate sufficient memory on the AD stack for outputs from all workers
   - setups all pointers for the operands and partials in advance
   - calculates its own work chunk assigned to the root
   - collects all results from all workers using the `gather` command. Here the function values and the gradients are collected in a combined way. All output is written directly onto the AD stack for which all pointers have been setup in advance
   - all results are registered in the AD stack

5. The workers are terminated with an MPI abort command which is sent whenever the root finishes.

### Rectangular Version

We might want to try to do a rectangular version first because the function signature is super simple.  From Stan, it will look like it has this signature:

Update from Sebastian: We should opt for a semi-mixed version which will give us all we ever need. That is, the input shall be rectangular, but the output will be a *ragged* array. The reason is simple: Padding `var` arrays would be very expensive while padding data is virtually for free given that data is only ever transmitted once. The *ragged* output can be easily transformed to a rectangular output if the output is indeed regular by using some `to_` conversion functions (convert to vector, then to matrix, done).

```
/**
 * Return sequence of results of applying function to parallel elements
 * of theta, x_r, x_i, so that for 1 <= n <= size(theta) 
 *
 *   map_rectangular(f, theta, x_r, x_i) = c(..., f(theta[n], x_r[n], x_i[n]), f(theta[n+1], x_r[n+1], x_i[n+1]), ...)
 */
real[] map_rect(F f, real[,] theta, real[,] x_r, int[,] x_i);
```

* *QUESTION: Should we call the function `apply()` rather than `map()`?* Lisp: (map f x);  Python: map(f, x);  R: sapply(f, x).


### Implementation

#### Top-Level Function

This is the only implementation of `map_rect`.  It delegates to `_mpi` or `_serial` implementation based on whether the `STAN_MPI_MAP` is defined.  

File `stan/math/prim/mat/functor/map_rect.hpp`:
```
template <typename F, typename T>
std::vector<Eigen::Matrix<T, -1, 1> >
map_rect(const F& f,
         const std::vector<Eigen::Matrix<T, -1, 1> >& theta,
         const std::vector<Eigen::VectorXd>& x_r,
         const vector<vector<int> >& x_i) {
#ifdef STAN_MPI_MAP
  return map_rect_mpi(f, theta, x_r, x_i);
#elif
  return map_rect_serial(f, theta, x_r, x_i);
#endif
}
```

#### General Serial Implementtion

This one will be used for all serial calls.  It applies the function to each element of the parallel argument arrays. 

File `stan/math/prim/mat/functor/map_rect_serial.hpp`:
```
template <typename F, typename T>
std::vector<Eigen::Matrix<T, -1, 1> >
map_rect_serial(const F& f,
                const std::vector<Eigen::Matrix<T, -1, 1> >& theta,
                const std::vector<Eigen::VectorXd>& x_r,
                const vector<vector<int> >& x_i) {
  check_match_sizes(theta, x_r);
  check_match_sizes(x_r, x_i);
  std::vector<Eigen::Matrix<T, -1, 1> > y(theta.size());
  for (size_t i = 0; i < theta.size(); ++i)
    y[i] = f(theta[i], x_r[i], x_i[i]);
  return y;
}
```



#### Primitive, Forward Parallel: `double`

This one can be templated to handle `double`.

File `stan/math/prim/mat/functor/map_rect_mpi.hpp`:

```
template <typename F>
std::vector<Eigen::VectorXd>
map_rect_mpi(const F& f,
             const std::vector<Eigen::VectorXd>& theta,
             const std::vector<Eigen::VectorXd>& x_r,
             const vector<vector<int> >& x_i);
```

There are lots of internal calcs that could use this, such as computing posterior summaries.

#### Reverse-mode Parallel: `var`

This is the one that will count for speed improvements.  It has to synchronize derivatives to reassemble the expression graph from the partials and result.  Matched by argument dependent lookup.

File `stan/math/rev/mat/functor/map_rect_mpi.hpp`:
```
template <typename F>
std::vector<Eigen::Matrix<var, -1, 1> >
map_rect_mpi(const F& f,
             const std::vector<Eigen::Matrix<var, -1, 1> >& theta,
             const std::vector<Eigen::VectorXd>& x_r,
             const vector<vector<int> >& x_i);
```

#### Forward parallel: `fvar<double>`, `fvar<fvar<double> >`

These need to be specialized to put an `fvar<double>` back together in terms of value and tangent of type `double`.  Same for `fvar<fvar<double> >` with value and tange of type `fvar<double>`.  Matched by argument dependent lookup.

File `stan/math/fwd/mat/functor/map_rect_mpi.hpp`:
```
template <typename F>
std::vector<Eigen::Matrix<fvar<double>, -1, 1> >
map_rect_mpi(const F& f,
             const std::vector<Eigen::Matrix<fvar<double>, -1, 1> >& theta,
             const std::vector<Eigen::VectorXd>& x_r,
             const vector<vector<int> >& x_i);

template <typename F>
std::vector<Eigen::Matrix<fvar<fvar<double> >, -1, 1> >
map_rect_mpi(const F& f,
             const std::vector<Eigen::Matrix<fvar<fvar<double> >, -1, 1> >& theta,
             const std::vector<Eigen::VectorXd>& x_r,
             const vector<vector<int> >& x_i);
```

These are very low priority.

#### Mixed parallel: `fvar<var>` and `fvar<fvar<var> >`

Mixed mode needs a custom implementation to handle `fvar<var>` out of the return values and do the right thing with value and tangent variables with type `var`.  This will be a huge win for computing things like Hessians.  Matched by argument dependent lookup.

File `stan/math/mix/mat/functor/map_rect_mpi.hpp`:
```
template <typename F>
std::vector<Eigen::Matrix<fvar<var>, -1, 1> >
map_rect_mpi(const F& f,
             const std::vector<Eigen::Matrix<fvar<var>, -1, 1> >& theta,
             const std::vector<Eigen::VectorXd>& x_r,
             const vector<vector<int> >& x_i);

template <typename F>
std::vector<Eigen::Matrix<fvar<fvar<var> >, -1, 1> >
map_rect_mpi(const F& f,
             const std::vector<Eigen::Matrix<fvar<fvar<var> >, -1, 1> >& theta,
             const std::vector<Eigen::VectorXd>& x_r,
             const vector<vector<int> >& x_i);
```

These would be particularly useful for Hessian calculations and for RHMC, but are low priority compared to reverse mode and primitive.