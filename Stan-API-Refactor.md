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


# Intermediate Steps from where (Cmd)Stan is at Now

## Refactor Command

Command needs lots of work to pull out functionality into separate services to ease testing and improve organization.  This has already begun with the initialization and some of the sampling steps, but needs to continue to  diagnostics, optimization, and sampler configuration.

The posterior analysis code needs similar work.

## Concrete Specificationss

We need concrete specifications for all of the configuration objects if we go with callbacks or for the protocol buffer or JSON schemas if we choose that route.
