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

#### message writer
    
* string: config [per config line] (not recoverable without structure) [DUPLICATED IN SAMPLE WRITER]
* string: (INFO) iteration # + sampling/warmup indicator 
* string: (WARN) non-fatal warning (e.g., rejection from sampler)
* string: (FATAL) fatal warning (e.g., can't find valid inits)
* string: timing  [DUPLICATED IN SAMPLE WRITER]


### Sample Writer

* string: Stan version numbers (one call per line; should be numeric)
* string: config [per config line] (not recoverable without structure)
* vector<string>: parameter names (e.g., CSV header) [exactly once]
* vector<double>: warmup values (once per iteration)
* double: adaptation stepsize
* vector<double> or matrix<double>: adaptation mass matrix
* vector<double>: sampling draws (one call per draw)
* string: timing (should be numeric and based on types)

### Diagnostic writer

* vector<string>: diagnosti quantity names (e.g., unconstrained parameters) (exactly once) [HEAVY DUPLICATION]
* vector<double>: warmup values (once per sampling iteration) [HEAVY DUPLICATION]
* vector<double>: sample values (once per sampling iteration) [HEAVY DUPLICATION]
* ... everything else from sample writer ... [COMPLETE DUPLICATION]
