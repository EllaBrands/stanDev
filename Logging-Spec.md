Following up on [this previous discussion](https://github.com/stan-dev/stan/wiki/Consolidated-Output-for-Sample,-Optimize,-ADVI,-and-Diagnose), moving forward we need to separate out I/O into information that is meant to be interactive and information that is meant to be passed to the interface programmatically, for example into a file or local R or Python environment.  

At the moment all I/O is awkwardly convolved in various ```interface_callback:::writer``` implementations.  For example, in MCMC implementations there are info and error writers as well as sample and diagnostic writers that are wrapped in the ```services::mcmc_writer``` class.

A logger would wrap all I/O that is destined for the user in realtime, typically written to the screen by one of the interfaces.  This would combine the info and error writers, and add even more resolution in the types of messages that can be written.  Following the log4j convection messages would be classified as

* ALL: print every message

* DEBUG: fine-grained app progress
* INFO: coarse app progress
* WARN: potentially harmful
* ERROR: recoverable error
* FATAL: cause app to abort

* OFF: no logging

Note that I'm not included TRACE as I don't think we'll need that level of granularity.

The logger itself would be easiest to use if implemented as a single class that could be passed around through the code, but there are various scopes of the logger and various ways to implement the different levels of output.

# Where to define the logger?

Should the logger be a wrapper that aggregates many ```interface_callback::writer`` instances, one for each level?

Or should the ```interface_callback::writer``` be generalized to implement the levels directly?  This latter approach would be somewhat awkward as ```interface_callback::writer``` instances are also used for file I/O which does not utilize these levels of output.

# How to implement logger levels?

Regardless of where the logger is defined, we have to define a specification for how to access each level.

For example, each level could be accessed by class methods, for example with ```operator::()``` as ```logger::warn("Message")``` or ```operator::<<``` as ```lower::warn() << "Message"```.

Alternatively, the levels could be specified with a prefixed token, as in ```logger << kWARN << "Message"```.


