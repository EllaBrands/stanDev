See:  http://discourse.mc-stan.org/t/proposal-for-consolidated-output/4263

for discussion.

NB: the text below is re-writing things from (Daniel Lee, Martin Modrák, and Krzysztof Sakrejda). 

## Problem

The current design has problems for moving forward:

1. There's an init writer, a sample writer, and a diagnostic writer + logger.  The sample writer gets algorithm parameters (mixed types) and model parameters (doubles) and is responsible for writing them all.  The diagnostic writer gets (for sampling) the momenta (doubles) for each parameter as well as the gradients (doubles) for each parameter.  The overall problem is that the separation of concerns is not clean (so, e.g. momenta and gradients are interleaved in the diagnostic writer) and some outputs don't have a clear place to go.  For example in CmdStan the diagonal of the mass matrix is output as a comment in the middle of the csv + comments output.  
2. The logger would be an ideal place to log additional info (such as optionally, as Martin has suggested, divergence-related information) but all logger output is text and it's creating extra work to, e.g., translate std::vector<double> -> text -> native R type.  The interfaces have to work too hard to implement basic functionality and they are diverging because of it.  
3. Output has to be manipulated within the algorithm implementations to fit right into the writers.  There's no reason for the algorithm to know about how its output is handled, it should just pass it on.

## Current goals:

1. Run a each Stan algorithm, from each interface, and produce output with minimal interface-specific code.
2. Maintenance. Be able to expand the output to accommodate new inference algorithms in addition to updating output for existing inference algorithms.
3. Be able to add new streams of diagnostic information easily (see e.g. [thread on adding ends of divergent transitions](http://discourse.mc-stan.org/t/getting-the-location-gradients-of-divergences-not-of-iteration-starting-points/4226/) )

## Future goals (so current non-goals):
1. Interoperability between interfaces. We should be able to run in one interface and perform analysis in another interface.  This will be a breaking change and is therefore Stan3 material.
2. Use subsets of the output. RStan and PyStan currently have features to only record certain values. CmdStan doesn't save the warmup by default. The interface implementation could be shared between RStan/PyStan/CmdStan and allow filtering. This is not a breaking change since default behavior can match current default behavior.

## Problem in detail

We currently have these inference algorithms:

- Sampling (13)
  1. fixed\_param
  2. hmc\_nuts\_dense\_e
  3. hmc\_nuts\_dense\_e\_adapt
  4. hmc\_nuts\_diag\_e
  5. hmc\_nuts\_diag\_e\_adapt
  6. hmc\_nuts\_unit\_e
  7. hmc\_nuts\_unit\_e\_adapt
  8. hmc\_static\_dense\_e
  9. hmc\_static\_dense\_e\_adapt
  10. hmc\_static\_diag\_e
  11. hmc\_static\_diag\_e\_adapt
  12. hmc\_static\_unit\_e
  13. hmc\_static\_unit\_e\_adapt
- Optimization (3)
  1. bfgs
  2. lbfgs
  3. newton
- (Experimental) Variational Inference (2)
  1. mean field
  2. full rank

Currently, the output approach works well for sampling since it's gotten most attention but not so much for optimization/ADVI.  For example ADVI has no obvious point density to calculate and currently outputs a confusing default value (NA or something?). This has posed difficulty with implementing new algorithms (e.g.- ADVI) and led to hacks in the interfaces or incomplete implementation of accessors for algorithm data.  All the interfaces have typed output (e.g.-error messages, mass matrix) that get converted to text and piped into the logger, which is not ideal since the interfaces have to take text and parse it to produce basic diagnostic output.

## What the output layer will provide to the algorithms:

A 'relay' object that has methods to dump output into _without conversion_, methods on the relay object will cover
1. key-value output `relay.kv<handler_type>(string key, T value)`
2. heterogeneous table output with:
  - Column types specified at template instantiation
  - Column names are specified at construction
  - Proper usage w.r.t. types is checked at construction
  - Proper usage w.r.t. number of columns checked at run-time
  - `relay.table<handler_type>(T1 v1, T2 v2, ...);`
  - (MAYBE): `relay.table<handler_type>(T v);` they do have to be 
    pushed in correct order but do not have to be collected prior to push.
3. homogenous table output with:
  - Column types specified at template instantiation
  - Column names are specified at construction
  - Proper usage w.r.t. types is checked at construction
  - Proper usage w.r.t. number of columns checked at run-time
  - requirement that an entire row is written at once.
  - `relay.array<handler_type>(std::vector<T>/Eigen::vector<T>)`
4. logger output (using the current logger), this is the text-output stream of last resort.

The algorithm should use the simplest one available and prefer to not modify output.  For example sampler parameters should be in a heterogeneous table if they are available near the same call site, key-value output can be used with irregularly-produced output, homogeneous tables are best for model parameters, and we should avoid separating output types with special suffixes (e.g.-the current situation with lp__, treedepth__, etc...).  The `handler_type` (consumer?) specifies types for the table/array and how different types are handled in the key/value call.  

## What the output implementation will provide to the interfaces

1. Interfaces should only have to write the `handler_type` classes for each array to cover
2. We should provide common handler_type classes for, e.g., file output
3. We should provide common handler_type classes for ignoring output

... more not done

## What algorithm implementations will have to provide to interfaces

Description of the required `handler_type`s

... more not done

-----------------------------------------------------------
Older stuff, hopefully merged above in an ok way.
-----------------------------------------------------------

## Alternative proposed solution

This is an alternative proposed by Martin Modrák. The main issue I have with the Proposed solution below is that it doesn't make it easy to add additional types of information - e.g. last points of divergent transitions, some per-step (not per-iteration) data etc. A new type of information would require a new function in the signature of the logging object, affecting all implementations (in particular most likely all interfaces).

In my proposal, the basic unit is a *data stream*. Each algorithm would provide a set of data streams. Those data streams will be piped through a single relay object. Consumers (interfaces) would register with the relay object to collect individual data streams. This will also nicely allow parameters passed to the interface to modify which information is gathered (e.g. per step information in NUTS might be handled only when necessary). Different algorithms will have different sets of recognized data streams, but otherwise the implementation would be identical. 

There would be multiple types of data streams:

1. Key-value => receives name-value pairs, order doesn't matter. Designed in particular for sending algorithm parameters
2. Table => Same as current stream of per-iteration data. Receives the names of columns and their data types first and then expects rows with this exact structure. It might also be useful to have a specific "indexing object" to be sent for each row, that would contain important info for possible filtering (iteration no. etc.) - otherwise it would be fragile to extract the necessary information from the individual rows.
3. Message log => For messages and errors. Might actually be implemented as a table (with columns like "Line","File","Level", "Message", ...). Although I wonder whether logging wouldn't be handled best by a dedicated logging library - I am not very familiar with the C++ ecosystem, but I guess there has to be something similar to log4j.

The set of streams for each algorithm will be defined by an `enum`. Alternatively, we might constrain that there would be only one key-value stream and only one message log stream, while multiple table streams would be allowed.

Each data stream will have its separate contract on data types, frequency, ordering etc. E.g. we might promise that the "algorithm parameters" stream will only be used before data start to be sent to "iterations" stream.

Some sketches of method signatures to make this clearer (ignoring the message log as this IMHO requires more thought):

Defining stream types:
```c++
class hmc_output_streams {
  enum key_value {algorithm_params, timing};
  enum table {iterations, divergences_end_points, unconstrained_params, rng_state };
  struct indexing {
     int iteration;
     bool is_warmup;
  }
}
```

consumers:
```c++
class key_value_consumer {
   virtual void consume_item(const string& key, const string& value) = 0;
   //Maybe allow strong-typed handling?
   virtual void consume_int(const string& key, int value) {
       consume_item(key, str_to_int(value));
   }  

   virtual void close() = 0;
}

template<class OutputStreams>
 class table_consumer {
    //Header defining functions, probably more combinations of type/arity needed
    virtual void add_double_column(const string& name) = 0;
    virtual void add_double_columns(const vector<string>& names) = 0;

    virtual void add_int_column(const string& name) = 0;
    virtual void add_string_column(const string& name) = 0;
   
    virtual void header_end() = 0;

    virtual void void row_start(const OutputStreams::indexing& index) = 0;
    virtual void consume_double(double value) = 0;
    virtual void consume_doubles(const std::vector<double>& values) = 0;
    virtual void consume_doubles(const Eigen::VectorXd& values) = 0;
    
    virtual void consume_int(int value) = 0;
    ...
    
    virtual void row_end() = 0;
    virtual void close() = 0;
 }
```

The relay object:
```c++
template<class OutputStreams>
 class output_relay {
    void set_key_value_consumer(OutputStreams::key_value stream_id, key_value_consumer& consumer);
    bool has_key_value_consumer(OutputStreams::key_value stream_id);

    //Would probably return some NUL consumer, if no consumer was registered
    key_value_consumer& get_key_value_consumer(OutputStreams::key_value stream_id); 

    void set_table_consumer(OutputStreams::table stream_id, table_consumer<OutputStreams::index>& consumer);
    bool has_table_consumer(OutputStreams::table stream_id);
    table_consumer<OutputStreams::index>& get_table_consumer(OutputStreams::table stream_id);
 }
```

Usage within the sampler:

```c++
void generate_transitions(..., output_relay<hmc_output_streams> &relay) {
   ...
   relay.get_key_value_consumer(hmc_output_streams::algorithm_params).consume_value("adapt_delta", delta);
   relay.get_key_value_consumer(hmc_output_streams::algorithm_params).close();
   ...
   table_consumer<hmc_output_streams::index> iterations_consumer = relay.get_table_consumer(hmc_output_streams::iterations);
   iterations_consumer.add_double_columns(<<model param names>>);
   iterations_consumer.add_double_column("energy__");
   iterations_consumer.add_int_column("divergent__");
   iterations_consumer.header_end();
   
   for (int m = 0; m < num_iterations; ++m) {
      ...
      iterations_consumer.row_start(hmc_output_streams::index(m));
      iterations_consumer.consume_doubles(<<values from sampler>>);
      iterations_consumer.consume_double(sampler.energy_);
      iterations_consumer.consume_int(sampler.divergent_);
      iterations_consumer.row_end()
   }
   iterations.close();
}

```

Optional streams withing transitions: 
```c++
bool base_nuts::build_tree(...) {
    if ((h - H0) > this->max_deltaH_) {
        this->divergent_ = true;
        // When efficiency is important, it may be reasonable to sorround the `consume` call in an if to avoid
        // gathering the info
        if(relay.has_table_consumer(hmc_output_streams::divergences_end_points)) {
          <<get the values from sampler>>
          relay.get_table_consumer(hmc_output_streams::divergences_end_points).consume_doubles(divergent_values);
        }
    }
  
}
```

CODA: maybe `consumer` / `consume` is not a great naming. Maybe `stream` / `send`?. Happy to hear suggestions.

## Proposed solution

I believe we should be handling sampling, optimization, and variational inference separately. The outputs should be interpreted differently (iterations in sampling are not like iterations in optimization or variational inference). Since most of our algorithms are in sampling, I'll outline what I think should happen using that as an example.

Once we have chosen a sampling algorithm, a lot of the structure of the output is determined. Once we know which Stan program and data, we now have most of the structure of the output. The last bit that determines the structure of the output is which bits of the output are actually saved. Here it is in more detail for the default sampling algorithm, hmc\_nuts\_diag\_e\_adapt:

- Adaptation information (determined by the algorithm + the model):
	- Stepsize: 1, type `double`
	- diagonal elements of inverse mass matrix: number of unconstrained parameters, type `double` 
- Data about the run (user-determined at runtime):
	- Number of warmup iterations
	- Number of sampling iterations
	- Which constrained parameters are being output (defaults to all)
	- Which of the generated quantities are being output (defaults to all)
	- Which unconstrained parameters are being output (defaults to none)
- For each iteration (this applies to both the header and each set of values), we can break it down into a few parts:
	- The class of algorithm determines there's an `lp__` value
	- This particular algorithm has these additional sampling variables (note: with stat hmc, a lot of these don't apply, with something like plain Metropolis Hastings, we'd have a different set of sampling variables):
		- `accept_stat__`
		- `stepsize__`
		- `treedepth__`
		- `n_leapfrog__`
		- `divergent__`
		- `energy__`
	- Constrained parameters that are saved
	- Generated quantities that are saved
	- Unconstrained parameters that are saved
- Elapsed time (for all algorithms)

Here's the current function signature for `hmc_nuts_diag_e_adapt`:
```c++
template <class Model>
int hmc_nuts_diag_e_adapt(Model& model, stan::io::var_context& init,
                          stan::io::var_context& init_inv_metric,
                          unsigned int random_seed, unsigned int chain,
                          double init_radius, int num_warmup,
                          int num_samples, int num_thin, bool save_warmup,
                          int refresh, double stepsize,
                          double stepsize_jitter, int max_depth,
                          double delta, double gamma, double kappa,
                          double t0, unsigned int init_buffer,
                          unsigned int term_buffer, unsigned int window,
                          callbacks::interrupt& interrupt,
                          callbacks::logger& logger,
                          callbacks::writer& init_writer,
                          callbacks::writer& sample_writer,
                          callbacks::writer& diagnostic_writer) {
```

Instead of just having `callbacks::writer& sample_writer` and `callbacks::writer& diagnostic_writer`, I'd suggest we move to splitting that into:

1. A writer for which algorithm has been chosen, perhaps the arguments to the algorithm, and maybe this can also stash the elapsed time.
2. A callback for the adaptation information. The one used here will only take the diagonal elements, but we'd also have one that accepts just the stepsize (for `unit_e`) and another that accepts the dense matrix (for `dense_e`).
3. A callback for things that pertain to sampling, so this should just accept `lp__`.
4. A callback for things that pertain to the `nuts` algorithms. This should accept `accept_stat__`, `stepsize__`, `treedepth__`, `n_leapfrog__`, `divergent__`, `energy__`. Other inference algorithms can have different things here.
5. A writer for all the constrained parameters. This can be configured to not write all values.
6. A writer for all the generated quantities.
7. A writer for the unconstrained parameters.

It would be on the interface to determine how these things are serialized. One implementation could be to recreate the CmdStan csv format, but I think we can move from that pretty easily.

### Concrete example: hmc\_nuts\_diag\_e\_adapt()


Proposed solution is to write an `hmc_writer` interface. This would be able to handle all of the 12 `hmc` related algorithms.

Here's what the `hmc_writer` interface would look like:
```
class {
  public:
    
};
```

Here's what the service method would look like:
```
template <class Model>
int hmc_nuts_diag_e_adapt(Model& model, stan::io::var_context& init,
                          stan::io::var_context& init_inv_metric,
                          unsigned int random_seed, unsigned int chain,
                          double init_radius, int num_warmup,
                          int num_samples, double stepsize,
                          double stepsize_jitter, int max_depth,
                          double delta, double gamma, double kappa,
                          double t0, unsigned int init_buffer,
                          unsigned int term_buffer, unsigned int window,
                          callbacks::interrupt& interrupt,
                          callbacks::logger& logger,
                          callbacks::hmc_writer& hmc_writer) {
```

Note:

- changing the writers to `hmc_writer`
- removing `num_thin`, `save_warmup`, and `refresh`; these are now controls of the `hmc_writer`

Here's how the flow of calling `hmc_writer` would happen through the `hmc_nuts_diag_e_adapt()` function:

- We don't need to save how `hmc_nuts_diag_e_adapt` was called. We can assume that the caller has access to exactly how it was called.
- Before the first iteration, it will call these functions in this order:
	- `initial_unconstrained_parameters(const std::vector<double>& unconstrained)`
	- `initial_sampler_state(...)`
	- `initial_sample_parameters(...)`?
		- `(log_prob__, accept_stat__)`
- Before each iteration, it will call these functions in this order:
	- `start_iteration(int iteration, bool warmup)`
    - `random_seed(int iteration, long, long)`
- Write out the sampler parameters. There are two options here:
	- After each time the sampler parameters change (with an adaptive warmup phase)
	- With each iteration
	- These are something like (not all of these together):
		- `step_size__`
		- `max_depth__`
		- the inverse mass matrix
	- I'll need to dig a little deeper, but there are really only a few things that change; what we need to be careful about is making sure everything that can change is captured. If we only write it out every change, then we can only start at each window boundary. If we write it out at every iteration, we can restart anywhere.
- After the competition of each iteration, it will call these functions in this order:
	- `sampler_parameters(int iteration, bool warmup, ...)` (this will have named and typed arguments)
		-  `(energy__)` 
		-  `(tree_depth__, n_leapfrog__, divergent__, energy__`)
	- `metric(int iteration, bool warmup, ...)`
	- `unconstrained_parameters(int iteration, bool warmup, const std::vector<double&> unconstrained)`
	- `transformed_parameters(int iteration, bool warmup, const std::vector<double&> unconstrained)`: maybe we can have a check function to be efficient about these functions
	- `constrained_parameters(int iteration, bool warmup, const std::vector<double&> unconstrained)`
	- `generated_quantities(int iteration, bool warmup, const std::vector<double&> unconstrained)`
- After the last iteration, we call the function:
	- `complete()` or something like that.

Some things that I haven't included:
- after warmup is finished, write out the time
- at complete, maybe write out the time	

If we guarantee the order that things are being written out to the `hmc_writer`, we can exactly create the CmdStan output.




## Reading the output

If we had things broken down into pieces like that, it'd be easy to do different things with them.

For parameter values (constrained parameters, generated quantities, unconstrained parameters), we want to compute:

- split R-hat
- mean
- sd
- quantiles
- n\_eff
Note: we don't need the adaptation information or the data about the run for us to compute those quantities.

For sampler parameters, we want to compute different things:

- max tree depth
- count the number of divergent transitions
- ... the list goes on and this is where we may be able to do better

We will want to read in the diagonal elements of the inverse mass matrix and the step size. And know the elapsed time. I think it'll be easier to reason if we were able to split apart the output.

Each interface could then decide how to serialize and deserialize the different output. We could provide a baseline implementation that could be used in CmdStan directly and would give the other interfaces a way to serialize in the same way as CmdStan.


--------------

Daniel, 7-8-2017: The text below is from before.

### *CURRENT* HMC Output

#### Message writer
    
* string: config [per config line] (not recoverable without structure) [DUPLICATED IN SAMPLE WRITER]
* string: gradient timing info (time per log density + gradient eval); plus projections for completion
* string: (INFO) iteration # + sampling/warmup indicator 
* string: (WARN) non-fatal warning (e.g., rejection from sampler)
* string: (FATAL) fatal warning (e.g., can't find valid inits)
* string: timing  [DUPLICATED IN SAMPLE WRITER]

#### Sample writer

* string: Stan version numbers (one call per line; should be numeric)
* string: config [per config line] (not recoverable without structure)
* vector<string>: parameter names (e.g., CSV header) [exactly once]
* vector<double>: warmup values (once per iteration)
* double: adaptation stepsize
* vector<double> or matrix<double>: adaptation mass matrix
* vector<double>: sampling draws (one call per draw)
* string: timing (should be numeric and based on types)

#### Diagnostic writer

* vector<string>: diagnostic quantity names (e.g., unconstrained parameters) (exactly once) [SOME DUPLICATION]
* vector<double>: warmup values (once per sampling iteration) [SOME DUPLICATION]
* vector<double>: sample values (once per sampling iteration) [SOME DUPLICATION]
* ... everything else from sample writer ... [COMPLETE DUPLICATION]


### *CURRENT* Optimization

#### Info writer (analogous to "Message" writer in HMC)

* string: config [per config line] (not recoverable without structure) [DUPLICATED IN OUTPUT WRITER]
* string: initial joint log probability (should be number)
* string: optimization progress header (should be vector<string>) [not just once, every 50 * refresh iterations]
* string: optimization progress values (should be vector<double> + notes string)
* string: note as to whether algorithm terminated normally (convergence message) or not (error msg) [broken down into two calls to strings]

#### Output writer (analogous to "Sample" writer in HMC)

* string: Stan version numbers (one call per line; should be numeric)
* string: config [per config line] (not recoverable without structure)
* vector<string>: parameter names (e.g., CSV header) [exactly once]
* vector<double>: parameter values (once per iteration OR just once at end depending on config to command)

#### Diagnostic writer (currently useless)

* ... repeats version numbers and config from output writer ... [DUPLICATE]


### *CURRENT* ADVI 

#### Message writer

* string: config [per config line] (not recoverable without structure) [DUPLICATED IN OUTPUT WRITER]
* string: this is advi (thank you very much); blank line; this is experimental
* string: gradient timing info (time per log density + gradient eval); plus projections
* string: "begin eta adaptation"
* string: iteration number (with same functions for formatting) [multiple lines, all say "Adaptation", hardcoded every 50]
* string: "success" (if successful)
* string: "all proposed stepsizes fail" (if fail to adapt, thrown as exception and then given to writer by top-level command dispatcher)
* string: "begin SGAscent (not Descent)"
* string: header for intermediate output (but not structured other than with spaces) (should be vector<string>)
* string: iterations for "main" ADVI algorithm output every so often based on refresh? (should be vector<double> plus string, where string is usually empty but may be some kind of note [are these documented somewhere?])
* string: "drawing N sample*s* from approximate posterior ... COMPLETED"


#### Parameter writer (analogous to "Sample" writer in HMC)

* string: Stan version number(s) (multiple lines)
* string: config [per config line] (not recoverable without structure) [DUPLICATED IN MESSAGE WRITER]
* vector<string>: csv header for "parameters" and the like
* string: adaptation information (eta, as two strings) [should be structured with number and variable]
* vector<double>: ADVI "solution" (mean parameters for normal approx, inv. transformed back to constrained) [exactly once]
* vector<double>: draws from variational approximation, inv. transformed back to constrained scale
* [ like optimization, no timing info written out ]

#### Diagnostic writer

* string: version number (multi line)
* string: config (multi line)
* string: header as comment
* vector<double>: iter, time-in-seconds, elbo (only every refresh or so often)


### *CURRENT* DIAGNOSE

#### Info writer (like "Message" writer for HMC)

* string: config [multiple lines]
* string: "test gradient mode"
* string: log prob = <value> [should have structure]
* string: "header" with idx, value, model finite diff, error
* string: "values" matching header [multiple lines, one per parameter]

#### Sample writer

[ exact dupe of info writer in terms of messages, output is to file with comments # at beginning of each line ]

#### Diagnostic writer

[ exact dupe of info writer in terms of messages, output is to file with comment # at beginning of each line ]


# BIG DESIGN DECISION

Do we write all this info out with structure or just as key-vals where the interface needs to know the set of possible keys?  



** IN PROGRESS BELOW HERE**

In general there are four "writers": 

- Info writer: key:value for per-run stuff, configuration and other single values (e.g.-adapted step size) always output complete configuration, is a bug otherwise, should be usable for reproducing a run, is a bug otherwise
- Progress writer: key-value with log level for progress messages, respects log4j log levels
- Output writer: csv-style output file, estimates and related diagnostic quantities
- Diagnostic writer: csv-style diagnostic file, internal versions of parameters and related diagnostic quantities, potentially respects log4j log levels

### *PROPOSED* HMC Output

#### Configuration writer
    
- key: string ("stan\_version\_major"), value: integer
- key: string ("stan\_version\_minor"), value: integer
- key: string ("stan\_version\_patch"), value: integer
- key: string ("model\_name"), value: string
- key: string ("method"), value: string
- key: string ("algorithm"), value: string
- key: string ("hmc:engine"), value: string
- key: string ("sampling:num\_samples"), value: integer
- key: string ("sampling:num\_warmup"), value: integer
- key: string ("sampling:save\_warmup"), value: integer, 0 = false, 1 = true
- key: string ("sampling:thin"), value: integer
- key: string ("nuts:adapt"), value: integer, 0 = false, 1 = true
- key: string ("nuts:gamma"), value: double
- key: string ("nuts:delta"), value: double 
- key: string ("nuts:kappa"), value: double
- key: string ("nuts:t0"), value: integer
- key: string ("nuts:init\_buffer"), value: integer
- key: string ("nuts:term\_buffer"), value: integer
- key: string ("nuts:window"), value: integer
- key: string ("nuts:max\_depth"), value: integer
- key: string ("hmc:metric"), value: integer
- key: string ("hmc:stepsize"), value: integer
- key: string ("hmc:id"), value: integer
- key: string ("input:data\_file"), value: string
- key: string ("input:init\_file"), value: string
- key: string ("hmc:seed"), value: integer (?)
- key: string ("output:configuration\_file"), value: string
- key: string ("output:samples\_file"), value: string
- key: string ("output:diagnostic\_file"), value: string
- key: string ("output:refresh"), value: integer
- key: string ("hmc:step\_size"), value: real
- key: string ("hmc:mass\_matrix"), value: real(s)
- key: string ("hmc:adaptation\_step\_size"), value: real
- key: string ("hmc:adaptation\_mass\_matrix"), value: vector/matrix of real
- key: string ("timing:gradient\_evaluation"), value: real
- key: string ("timing:estimate"), value: real, definition: (s) per 1000 transitions, 10 leapfrog steps per transition (s)")

#### Progress writer:
- key: string ("iteration"), value: integer
- key: string ("WARN"), value: string
- key: string ("FATAL"), value: string

#### Sample writer, NOT key-value, order matters:

- parameter names: value: vector of strings
- warmup values: value: vector of real 
- sampling draws: value: vector of real 

#### Diagnostic writer, NOT key-value, order matters:

- diagnostics quantity names: value: vector of strings
- warmup values: value: vector of reals 
- sampling draws: value: vector of reals 




### *PROPOSED* Optimization

#### Info writer
- key: string ("stan\_version\_major"), value: integer
- key: string ("stan\_version\_minor"), value: integer
- key: string ("stan\_version\_patch"), value: integer
- key: string ("model\_name"), value: string
- key: string ("method"), value: string
- key: string ("algorithm"), value: string
- key: string ("optimization:init\_alpha"), value: real
- key: string ("optimization:tol\_obj"), value: real
- key: string ("optimization:tol\_rel\_obj"), value: real
- key: string ("optimization:tol\_grad"), value: real
- key: string ("optimization:tol\_rel\_grad"), value: real
- key: string ("optimization:tol\_param"), value: real
- key: string ("optimization:history\_size"), value: integer
- key: string ("optimization:max\_iterations"), value: integer
- key: string ("optimization:save\_iterations"), value: integer, 0 = false, 1 = true
- key: string ("id"), value: integer
- key: string ("input:data\_file"), value: string
- key: string ("input:init\_file"), value: string
- key: string ("seed"), value: integer (?)
- key: string ("output:estimate\_file"), value: string
- key: string ("output:diagnostic\_file"), value: string
- key: string ("output:refresh"), value: integer
- key: string ("optimization:initial\_log\_probability"), value: real, definition: initial log joint probability

#### Progress writer
- key: string ("iteration"), value: integer
- key: string ("WARN"), value: string
- key: string ("FATAL"), value: string
- key: string ("INFO"), value: string
- key: string("outcome"), value: string (e.g.-"converged")

#### Sample writer, NOT key-value, order matters:

- parameter names: value: vector of strings
- warmup values: value: vector of real 
- sampling draws: value: vector of real 

#### Diagnostic writer, NOT key-value, order matters:

- diagnostics quantity names: value: vector of strings
- warmup values: value: vector of reals 
- sampling draws: value: vector of reals 



### *PROPOSED* ADVI 

#### Info writer


#### Message writer
- key: string ("iteration"), value: integer
- key: string ("WARN"), value: string
- key: string ("FATAL"), value: string
- key: string ("INFO"), value: string
- key: string("outcome"), value: string (e.g.-"converged")


#### Sample writer, NOT key-value, order matters:

- parameter names: value: vector of strings
- warmup values: value: vector of real 
- sampling draws: value: vector of real 

#### Diagnostic writer, NOT key-value, order matters:

- diagnostics quantity names: value: vector of strings
- warmup values: value: vector of reals 
- sampling draws: value: vector of reals 



### *PROPOSED* DIAGNOSE

#### Info writer (like "Message" writer for HMC)

#### Sample writer

#### Diagnostic writer
