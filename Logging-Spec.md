# Logging Recommendations

These are recommendations after looking at what we currently do to pass messages. We're trying to balance consistency across messages, ease of implementation, ease of maintenance, and enough flexibility to do enough in the future.

From the interfaces, CmdStan and PyStan implement the easy thing. RStan reads messages and tries to group them by string. I think we can plan around that.

Here are the different ways the messages are currently being generated:

1. We are writing static strings.
2. We write blank output to control spacing.
3. We catch `std::exception& e`, write a static string, then `e.what()`.
4. We are generating more complex output using `std::stringstream`. We write the contents of the `stringstream` using the `str()` method. Most often than not, we're using `stringstream` to convert `double`s to string.
5. Some of our legacy code is writing to streams. We are passing a `stringstream` and writing the contents.
6. We write an experimental algorithm message.

## Design / prototype

After looking at how we currently use the messages, my suggestion would be to have the messages from Stan's services be treated as a unit. Services are passing whole messages to be output or not. The interfaces are responsible for displaying the correct level. We'll have to decide how labels get levels, but the first cut, we can assign everything as the same level. 

```C++
class Logger {
  public:
    void debug(std::string);
    void debug(std::stringstream);

    void info(std::string);
    void info(std::stringstream);

    void warn(std::string);
    void warn(std::stringstream);

    void error(std::string);
    void error(std::stringstream);

    void fatal(std::string);
    void fatal(std::stringstream);
};
```

At some point, perhaps we can add a little more resolution: either adding a `std::string` identifier (something that can be used for programmatic filtering), more of a `printf()` or other string handling format, or even multiple arguments with different types.

## From the services

Minimal change. We just need rules determining which messages go where.

## From the interfaces

CmdStan and PyStan are easy. I need to dig into RStan.


# General Logging discussion

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

## Where to define the logger?

Should the logger be a wrapper that aggregates many ```interface_callback::writer`` instances, one for each level?

Or should the ```interface_callback::writer``` be generalized to implement the levels directly?  This latter approach would be somewhat awkward as ```interface_callback::writer``` instances are also used for file I/O which does not utilize these levels of output.

## How to implement logger levels?

Regardless of where the logger is defined, we have to define a specification for how to access each level.

For example, each level could be accessed by class methods, for example with ```operator::()``` as ```logger::warn("Message")``` or ```operator::<<``` as ```lower::warn() << "Message"```.

Alternatively, the levels could be specified with a prefixed token, as in ```logger << kWARN << "Message"```.


