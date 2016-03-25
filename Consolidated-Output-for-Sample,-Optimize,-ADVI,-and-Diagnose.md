Ideally, nothing will be order sensitive other than the order of iterations.

## log4j log levels:

* ALL:  print every message

* TRACE(int granularity): very fine-grained progress
* DEBUG: fine-grained app progress
* INFO: coarse app progress
* WARN: potentially harmful
* ERROR: recoverable error
* FATAL: cause app to abort

* OFF: no logging


## *CURRENT* HMC Output

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


## *CURRENT* Optimization

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


## *CURRENT* ADVI 

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


## *CURRENT* DIAGNOSE

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


   

