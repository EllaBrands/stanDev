With the first successful MPI prototype (https://github.com/stan-dev/cmdstan/tree/feature/proto-mpi/examples/mpi), let's start to design it here.

The basic idea of parallelism using the message passage interface is that we can use autodiff just as is, since MPI will start *independent* processes. All what MPI does is to organize the exchange of messages between the different processes.

# Requirements

- guess what...you need MPI! Using macports this can be installed on macOS with `sudo port install openmpi`. This will install a MPI compiler and the mpirun command.
- `boost.mpi` & `boost.serialization` libraries installed as binaries and hooked into the Stan build system

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

# Open points

- Name for the new function?
- Do we want to enable multiple `map` calls in a Stan program or just allow a single one?
- Is the ragged array construct what we want to use (it really is a workaround)?
- How is the backend configured? At compile time?
- Backends we need are: MPI, serial, others?

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

```
/**
 * Return sequence of results of applying function to parallel elements
 * of theta, x_r, x_i, so that for 1 <= n <= size(theta) 
 *
 *   map_rectangular(f, theta, x_r, x_i)[n] = f(theta[n], x_r[n], x_i[n])
 */
vector[] map_rect(F f, vector[] theta, vector[] x_r, int[,] x_i);
```

The function `f` itself is the same as in the ragged version.

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