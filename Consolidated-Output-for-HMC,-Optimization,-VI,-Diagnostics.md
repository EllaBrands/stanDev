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

* vector<string>: diagnostic quantity names (e.g., unconstrained parameters) (exactly once) [HEAVY DUPLICATION]
* vector<double>: warmup values (once per sampling iteration) [HEAVY DUPLICATION]
* vector<double>: sample values (once per sampling iteration) [HEAVY DUPLICATION]
* ... everything else from sample writer ... [COMPLETE DUPLICATION]


## *CURRENT* Optimization

No diagnostic output.

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


## ADVI 


## DIAGNOSE




   


