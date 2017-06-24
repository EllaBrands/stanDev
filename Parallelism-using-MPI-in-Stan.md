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

# Open points

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

The current proposal is to define a `map` function which takes as arguments

- a worker function which performs a chunk of work. The signature of that function shall be
```
real[] foo(real[] params, real[] x_r, int[] x_i)
```

- we also need a integer array `int[] f_out` which encodes the ragged structure of the return values of the worker function. The user anyway must know the output structure himself to be able to make sense of any output.

- ragged arrays of parameters and data (real and int)

In pseudo code this will look like

```
real[] map(F f, int[] f_end,
           real[] param, int[] param_end,
           real[] x_r, int[] x_r_end,
           int[] x_i, int[] x_i_end) {
  ... validate all ends are same size, in bounds, ordered
  real[f_end[size(f_end)]] y;
  int f_start = 1;
  int param_start = 1;
  int x_r_start_ = 1;
  int x_r_start = 1;
  for (i in 1:size(param_end) *in parallel*) {
    y[f_start:f_end[i]] = f(param_start[param_start:param_end[i]], 
                            x_r[x_r_start:x_r_end[i]],
                            x_i[x_i_end:x_i_end[i]]);
    param_start = param_end[i] + 1;
    x_r_start = x_r_end[i] + 1;
    x_i_start = x_i_end[i] + 1;
    f_start = f_end[i] + 1;
  }
  return y;
}
```

The *in parallel* section will proceed in the steps

1. ship all parameters block-wise to the worker nodes. I.e. the blocking is fixed by the order in advance. This ensures that the communication is done in a single go at once which can be implemented with the MPI `scatter` command in MPI.

2. Each worker performs their packet of work. This includes function evaluation and gradient computation.

3. All results are collected using the MPI `gather` command. This is done such that the results are directly written to the AD memory stack of the root node.

4. On the root node, the results are injected into the AD tree with the `precomputed_gradients` approach using pointers to avoid unnecessary copies.

The above procedure assumed that the data has been shipped once to the workers. This is doable given that we can detect that the data is in scope of the `map` construct.

# Notes on the prototype

The `cmdstan` implementation works roughly as follows:

1. A modified `main.cpp` is used. The MPI environment is initialized in the `main-mpi.cpp` and the `env.abort()` is called on the main process to shutdown all workers when the root terminates.

2. The root and the workers start normally. All processes call at the end of the `transformed data` block a `setup_mpi_function`.

3. The `setup_mpi_function` determines the `f_out` (ragged structure of the function outputs) and behaves differently on  the different nodes
   - workers go into an infinite loop where they wait for packets of parameters from the root; in addition the workers subset the data to the set of data which is allocated the the worker
   - the root: here the function returns with a data structure which maps each job to a worker id and the output sizes per job

4. only on the root node the `run_mpi_function` is ever called in the `transformed parameters` block which does
   - sent blockwise the parameters to the workers using the MPI scatter command
   - allocate sufficient memory on the AD stack for outputs from all workers
   - setups all pointers for the operands and partials in advance
   - calculates its own work chunk assigned to the root
   - collects all results from all workers using the `gather` command. Here the function values and the gradients are collected in a combined way. All output is written directly onto the AD stack for which all pointers have been setup in advance
   - all results are registered in the AD stack

5. The workers are terminated with an MPI abort command which is sent whenever the root finishes.



