Stan C++ will provide an API to the interfaces, offering services for optimization, sampling, and the like.  Our current goal takes this even further, encapsulating a full Stan process into a single function callable from any external interfaces.

- _Services_ will refer to C++ functions that the Stan API provides.  These might include small tasks like initialization, larger tasks like warmup and sampling, and the monolithic command function. 
 
-  _Interface elements_ are manipulated by the external interfaces to read and write data and configuration information.  

An unrelated service the Stan C++ API will provide is Rhat and ESS calculations.  The plan is to spec out the basic commands for running first and worry about posterior analysis later, as it is an independent problem.  

The overarching goal is to create a single `command` service in the API that requires at least the following interface elements, all of which are provided as part of the call to the service or returned by the service.  The following definitions assumes they will all be callbacks (i.e., C++ objects defined by the interfaces that provide methods that the services can call):

- _Interrupter_: This allows for the interface to interrupt the execution of command, for example in between MCMC transitions.  This is required if there is to be a way for the interfaces to halt the process using a control sequence.  This may include iteration number, chain ID for MCMC, a flag to indicate whether it's a warmup iteration, sampling iteration, or optimization iteration.

- _VarWriter_: This accepts variable/value pairs from Stan C++ and stores or outputs them as desired.  Writers break down into ones that apply
    - per iteration: model parameters and algorithm parameters (e.g., accept rate, n_divergent, etc.)
    - per adaptation event: adaptation parameters

- _VarReader_: This callback provides a means for Stan C++ to read a list of named values, and will be used for
    - data: providing data to the model
    - initialization: providing initial values to an algorithm
    - algorithm config: providing things like mass matrices and step sizes for algorithms

- _Output Streams_:  These can play the role of `std::cout` and `std::cerr` (if necessary).  These may not be necessary if the interfaces do all of their output (such as refreshing the iteration number) themselves using the callbacks.

- _Config_ :  This element will provide values for configuration, such as choice of method (sampling, optimizing, diagnosing), algorithm (dependent on method, such as HMC or L-BFGS), engine (NUTS vs. basic HMC), and finer-grained config such as random number seed, etc.  As things stand now, elements of config will determine which of the other elements are actually used (e.g., optimization will not accept a mass matrix argument)

The signature of command using callbacks might then be declared as follows.

```
template <class Program, class VarReader, class Interrupt, 
          class AdaptationWriter, class IterationWriter>
void
command(Program& program,
        const VarReader& data,
        const VarReader& inits,
        Interrupt& interrupter,
        AdaptationWriter& adapt_writer,
        IterationWriter& it_writer,
        std::ostream& output_stream,
        std::ostream& error_stream,
        const stan::command::config& config);
```

One issue that's already come up is compile time with all of these being templated concepts rather than abstract base classes.  

- _Templated Concepts_:  Very flexible, no virtual function call overhead

- _Virtual Base Classes_:  Much faster to compile.  Easier to understand because the contract is in the C++ object language in the base class
    - Doing this for the model would require the model to be constructed before being sent into the command, and would require either the data reader to be provided to construct the model or a new method to be included in the model to set the data variables (this would be no problem at all to code, by the way).

- _I/O Streams_: These could simply be eliminated if the interfaces took responsbility for their own output.  For example, rather than writing the iteration (and chain?) number with a certain refresh rate, the interrupt callback could be provided with the iteration number, chain ID, and a flag indicating whether it's a warmup MCMC iteration, sampling MCMC iteration, or an optimization iteration.  Eliminating the streams seems like the easiest way to implement a quiet mode in an interface-dependent way.

- _Configuration_:  This provides a means for the services to read the configuration information provided by the interfaces to run a service.  

## Config by Factory or Instance?

Should the interfaces create a variable reader, writer, and callback handling factories and let `command` construct the actual instances, or should the interfaces just create the objects externally and then pass those into `command`?  There doesn't seem to be much benefit to factories here, as it's unlikely that an interface will want to pass in a factory that could be configured by the service.

## Alternative:  Protocol Buffers or JSON?

An alternative to callbacks is to replace one or more of the above elements with protocol buffer or JSON streams (either input or output).  

- _Pros_:  Human readable, less C++ coding, single data structure usable across interfaces as files.

- _Cons_:  Lack of flexiblity, compile-time error checking, slow, memory-intensive, introduce dependencies on further libraries.

Protocol buffers seem more evolved (and more heavyweight in terms of interface) than JSON in that they allow data type schemas and provide a binary format (which would reduce memory consumption while removing human readability).  They also have more mature and faster implementations than JSON.


## Intermediate Steps from where (Cmd)Stan is at Now

#### Refactor Command

Command needs lots of work to pull out functionality into separate services to ease testing and improve organization.  This has already begun with the initialization and some of the sampling steps, but needs to continue to  diagnostics, optimization, and sampler configuration.

The posterior analysis code needs similar work.

#### Concrete Specificationss

We need concrete specifications for all of the configuration objects if we go with callbacks or for the protocol buffer or JSON schemas if we choose that route.

## Existing CmdStan Configurations

### Monolith vs. Modules?

The big question here is whether we want the interfaces to provide a single function, such as `stan()`, that takes a monolithic configuration (aka <a href="http://en.wikipedia.org/wiki/God_object">god object</a>, a notorious anti-pattern), or whether to make them modular (the source of all that's good in software engineering), such as nuts(), ehmc(), lbfgs(), diagnose(), ... [Bob's editorializing].  

A further advantage to breaking these down is that there's less high-level spec in the calls (e.g., `sample algorithm=hmc engine=nuts`) because it's implied by the top-level command.  Also, the consistency checking is simpler because the arguments can all be flattened out rather than being hierarchical.  

Yet another advantage is that when we introduce new methods or algorithms or engines, the change will be localized to that function call and won't impact other commands.  

The major disadvantage is that it provides more ways to introduce inconsistencies on the interface side than a monolothic command.

### Commands

The following shows the CmdStan configurations, which are implemented monolithically in Stan 2, but are documented in the following sub-pieces because I (Bob) found it easier to understand.  The parameters with the same name across the "different" commands are all the same and could use the same validation.

#### Euclidean NUTS

```
sample 
  algorithm=hmc
    engine=nuts
      [max_depth=<int>]
    [metric={unit_e,diag_e,dense_e}]
    [stepsize=<double>]
    [stepsize_jitter=<double>]
  [num_samples=<int>]
  [num_warmup=<int>]
  [save_warmup=<boolean>]
  [thin=<int>]
  [adapt
    [engaged=<boolean>]
    [gamma=<double>]
    [delta=<double>]
    [kappa=<double>]
    [t0=<double>] ]
  [data file=<string>]
  [init=<string>]
  [random seed=<int>]
  [output
    [file=<string>] 
    [diagnostic_file=<string>]
    [refresh=<int>] ]
```

#### Euclidean HMC without NUTS

This only differs from NUTS in the engine argument and sub-argument, so if any two are going to be combined, it'd be these two.

```
sample 
  algorithm=hmc
    engine=static
      [int_time=<double>]
    [metric={unit_e,diag_e,dense_e}]
    [stepsize=<double>]
    [stepsize_jitter=<double>]
  [num_samples=<int>]
  [num_warmup=<int>]
  [save_warmup=<boolean>]
  [thin=<int>]
  [adapt
    [engaged=<boolean>]
    [gamma=<double>]
    [delta=<double>]
    [kappa=<double>]
    [t0=<double>] ]
  [data file=<string>]
  [init=<string>]
  [random seed=<int>]
  [output
    [file=<string>] 
    [diagnostic_file=<string>]
    [refresh=<int>] ]
```

#### L-BFGS Optimization

```
optimize
  algorithm=lbfgs
    [init_alpha=<double>]
    [tol_obj=<double>]
    [tol_rel_obj=<double>]
    [tol_grad=<double>]
    [tol_rel_grad=<double>]
    [tol_param=<double>]
    [history_size=<int>]
  [iter=<int>]
  [save_iterations=<boolean>]
  [data file=<string>]
  [init=<string>]
  [random seed=<int>]
  [output
    [file=<string>]
    [diagnostic_file=<string>]
    [refresh=<int>] ]
```

#### BFGS Optimization

Just like L-BFGS other than algorithm and history config.

```
optimize
  algorithm=bfgs
    [init_alpha=<double>]
    [tol_obj=<double>]
    [tol_rel_obj=<double>]
    [tol_grad=<double>]
    [tol_rel_grad=<double>]
    [tol_param=<double>]
  [iter=<int>]
  [save_iterations=<boolean>]
  [data file=<string>]
  [init=<string>]
  [random seed=<int>]
  [output
    [file=<string>]
    [diagnostic_file=<string>]
    [refresh=<int>] ]
```

#### Newtonian Optimization

Like BFGS and L-BFGS, but with different algorithm and no algorithm-specific parameters.

```
optimize
  algorithm=newton
  [iter=<int>]
  [save_iterations=<boolean>]
  [data file=<string>]
  [init=<string>]
  [random seed=<int>]
  [output
    [file=<string>]
    [diagnostic_file=<string>]
    [refresh=<int>] ]
```

#### Diagnostics

So far, this only provides tests of the gradient.  

```
diagnose 
  [test=gradient]
    [epsilon=<real>]
    [error=<real>]
  [data file=<string>]
  [init=<string>]
  [random seed=<int>]    
```

## Other Interface Functions

#### Compiling (and Linking) Stan Programs

CmdStan uses `make` directly to compile models from files.  

RStan provides a function to compile (and dynamically link) a Stan model.  The `stan()` command for sampling allows the model to be specified as a compiled object, a text string, or a file, with the latter two being compiled as part of the `stan()` command.  RStan 2.6.3 is moving to building make-like behavior into the compilation with `stan()` (and maybe within compilation).  RStan requires optimization to use a precompiled model.  I don't know if models can be serialized to disk explicitly now or if that will only happen as part of make-like `stan()` function.

#### Fit Object

RStan produces a `stanfit` object as a result of the `stan()` command that is very much like the output of `glm()` in R --- it encapsulates a whole bunch of what went into the command, including the compiled model and the output.

#### Extracting Output

RStan provides a means of extracting the draws from a fit object so that they can be manipulated within R.

#### Posterior Analysis

The interfaces RStan and PyStan implement their own versions of ESS and R-hat.  These should be provided as services by Stan C++ in the future.  Should the print formats be the same?  Should they be minimal in the way Andrew likes them or complete as everyone else seems to prefer?  Should we use scientific notation or let R (or Python) do its print "magic"?

#### Graphical Display

RStan provides graphical displays of various things like traceplots.  This functionality may all be moving over to shinyStan.

#### Log Prob and Constrained/Unconstrained Transformations

RStan and PyStan provide a means of accessing the compiled model's log prob and gradient functions, as well as a means of translating from constrained to unconstrained parameters and back again.