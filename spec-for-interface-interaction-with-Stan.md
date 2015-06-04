**DEPRECATED**

New page: [User Interface Guidelines for Developers](https://github.com/stan-dev/stan/wiki/User-Interface-Guidelines-for-Developers)


This is where the spec for the interaction with Stan should go.

I believe the interfaces, {*}Stan, should provide a single instantiated concept that has:

1. A mechanism for transmitting messages at different levels. We currently only have two message streams, output and error, but we're not careful about how we use them. For the time being, most of these can just not be used.
    1. error
    2. warn
    3. info
    4. debug
    5. trace

2. A mechanism for getting draws back from Stan. Right now it's implemented through recorders that assume that it can do actions like push in a draw and write messages to a csv file. CmdStan just implements a way to stream to file. RStan implements a recorder that can 1. accept all variables of a draw or optionally accept a subset of variables and save that to memory and/or 2. stream to file.

3. A mechanism for getting diagnostic draw back from Stan. Right now, CmdStan streams that to file or ditches the diagnostic samples. RStan does the same. It might makes sense that this is combined with 2.

4. A callback for handling ctrl-c within interfaces. CmdStan does nothing. RStan implements a break using Rcpp, I think.

5. warmup iterations

6. pseudo random number generator

That's what I can think of so far. I'll try to come up with a design that captures what the interface needs to be.
