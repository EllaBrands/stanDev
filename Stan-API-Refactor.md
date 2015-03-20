Stan C++ will provide an API to the interfaces, offering services for optimization, sampling, and the like.  Our current goal takes this even further, encapsulating a full Stan process into a single function callable from any external interfaces.

Going forward, 

- _Services_ will refer to C++ functions that the Stan API provides.  These might include small tasks like initialization, larger tasks like warmup and sampling, and the monolithic command function.  
-  _Interface elements_ are C++ functors that external interfaces define and pass into services.  These are mostly responsible for translating I/O between an interface and Stan.

An unrelated service the Stan C++ API will provide is Rhat and ESS calculations.

As of this moment the goal is to create a single `command` service in the API that requires at least the following three interface elements, all of which take the form of callbacks, and all of which are provided as part of the call to the service (i.e., part of the config):

- _interrupter_: This allows for the interface to interrupt the execution of command, for example in between MCMC transitions.  This is required if there is to be a way for the interfaces to halt the process using a control sequence.

- _writer_: This element accepts events from Stan and stores or outputs them as desired.  Example events will include events
    - per iteration: model parameters and algorithm parameters (e.g., accept rate, n_divergent, etc.)
    - per adaptation event: adaptation parameters
The writer could be (and probably should be) broken into several smaller interfaces.

- _var_context_factory_: This element is responsible for providing a way for Stan C++ to read data and/or initial values.  Could initial values and data be part of the config object? (@ariddell))

- _config_:  This element will provide values for configuration, such as choice of method (sampling, optimizing, diagnosing), algorithm (dependent on method, such as HMC or L-BFGS), engine (NUTS vs. basic HMC), and finer-grained config such as random number seed, etc.

The signature of command might then look like

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
        const stan::command::config& config)
```

One issue that's already come up is compile time with all of these being templated concepts rather than abstract base classes.  If they were made abstract base classes, it would be much faster to compile, but it would present the problem that the model (i.e., Stan program) would have to be instantiated with data before being passed in, unless we reorganized the generated model code to read data (and construct transformed data) through a method rather than through the constructor.  It would also add virtual function resolution overhead, but that's almost certainly not going to be noticeable given how complicated all these functions are.

There's also the question of whether we want to pass bare streams for errors and or output or leave that up to the interfaces to produce based on iteration-level callbacks and capturing exceptions.

# Intermediate Steps

## Refactor Command

Command needs lots of work to pull out functionality into separate services to ease testing and improve organization.  This has already begun with the initialization and some of the sampling steps, but needs to continue to  diagnostics, optimization, and sampler configuration.

The posterior analysis code needs similar work.

## Spec Writer Interface Element

We need to define a writer spec for adaptation and iterations and then update all internal Stan code to use a writer element instead of std::cout and std::cerr.  This will have the added benefit of removing std::cout and std::cerr from Stan completely.

## Spec Reader Interface Element

A bit more work needs to be done for the input readers.  Should the interfaces create a `var_context_factory` and let `command` be responsible for  construct the actual `var_context` objects, or should the interfaces just create a few `var_context` objects externally and then pass those into `command`?

Another option to consider is a more general purpose approach to the problem `var_context` is intended to solve. Protocol Buffers, for example, offer a binary serialization format for structured data precisely like `var_context` (repeated doubles/ints and strings). Not having to implement a C++ callback like `var_context` in the first place  may make it easier to develop an interface.

Whatever is done here, it can be reused to specify all of 
- data input
- initialization input
- fixed mass matrix and stepsize inputs

## Spec the Configuration Objects

One we've specified the inputs, we have to organize the configuration and decide how the interfaces will configure `command`.  A string?  A C++ class shared by all interferes, or do we just specify a policy and go with another interface  element here?

# The Big Question

A major issue to resolve is whether to just toss the idea of having callbacks and instead have all of the readers just provide protocol buffers and have all of the writers just receive a protocol buffer stream (assuming a protocol buffer can be streamed --- otherwise it would have to be buffered).

