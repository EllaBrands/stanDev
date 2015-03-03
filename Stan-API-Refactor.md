Ideally Stan would serve as a well-designed API to the interfaces,
offering services for optimization, sampling, and the like.  Our
current goal takes this even further, encapsulating a full Stan
process into a single function callable from any external interfaces.

Going forward, _services_ will refer to C++ functions that the Stan
API provides.  These might include small tasks like initialization,
larger tasks like warmup and sampling, and the monolithic command
function.  On the other hand, _interface elements_ are C++ functors
that external interfaces define and pass into services.  These are
mostly responsible for translating I/O between an interface and
Stan.

As of this moment the goal is to create a single `command` service
in the API that requires three interface elements

- callback: This allows for the interface to interrupt the execution 
of command, for example in between MCMC transitions.

- writer: This element accepts messages from Stan and outputs them
to the interface as desired, for example writing to a file or outputting
to a screen.

- var_context_factory: This element is responsible for storing input
values, such as data and initialization.  

The signature of command might then look like

    template <class Model, class Callback, class Writer, class VarContext>
    command(Model& model,
            Callback& callback,
            Writer& writer,
            VarContext& var_context,
            config_type& config)

# Intermediate Steps

There is, unsurprisingly, lots to do.

## Refactor Command

Command needs lots of work to pull out functionality into separate services
to ease testing and improve organization.  This has already begun with the
initialization and some of the sampling steps, but needs to continue to 
diagnostics, optimization, and sampler configuration.

## Spec Writer Interface Element

We need to define a writer spec and then update all internal Stan code to
use a writer element instead of std::cout and std::cerr.  This will have
the added benefit of removing std::cout and std::cerr from Stan completely.

## Spec Input Interface Element

A bit more work needs to be done for the inputs.  Should the interfaces
create a `var_context_factory` and let `command` be responsible for 
construct the actual `var_context` objects, or should the interfaces
just create a few `var_context` objects externally and then pass those
into `command`?

Another option to consider is a more general purpose approach to the problem `var_context` is intended to solve be viable? Protocol Buffers, for example, offer a binary serialization format and there is good R and Python support for Protocol Buffers. Not having to wrap a C++ class like `var_context` in the first place seems like a very good outcome.

## Spec the Configuration Objects

One we've specs the inputs we have to organize the configuration and decide
how the interfaces will configure `command`.  A string?  A C++ class shared
by all interferes, or do we just specify a policy and go with another interface 
element here?

One example of a challenging configuration to setup is when a user provides a vector or matrix
of values that should serve as initial values for the sampler. Serializing and deserializing these data structures into strings may be an option. Protocol Buffers or JSON or XML also seem like viable candidates.