## Problem

As I'm working through how the output is generated from within Stan, I'm seeing some of the technical difficulties we have with our current design (single writer trying to handle all types of output). I'll try to outline what we're trying to do, then describe what I think could work for us.

These are some of our goals:

1. Run a specific inference algorithm from a particular Stan interface and produce output.
2. Take the output and perform analysis afterwards.
3. Interoperability between interfaces. We should be able to run in one interface and perform analysis in another interface.
4. Use subsets of the output. RStan and PyStan currently have features to only record certain values. CmdStan doesn't save the warmup by default. We should continue to support this behavior.
5. Maintenance. Be able to expand the output to accommodate new inference algorithms in addition to updating output for existing inference algorithms.

What we're missing:

- Momento. With Mitzi's latest fixes with the metric and the two-argument random seed, we should be able to stop and restart.


## Inference Algorithms

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

Currently, we only have one way of reading the output, which assumes that the output was sampled. In practice the different classes of inference algorithms do not even treat the posterior density consistently.  For example the log density for sampling is calculated for the point representing draw, whereas for VI there is no one obvious point to calculate the density at.  This has posed difficulty with implementing new algorithms (e.g.- ADVI) and led to hacks in the interfaces or incomplete implementation of accessors for algorithm data.  

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

