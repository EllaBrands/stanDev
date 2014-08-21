This is where the spec for the interaction with Stan should go.

I believe the interfaces, {*}Stan, should provide a single instantiated concept that has:
1. A mechanism for transmitting messages at different levels. We currently only have two message streams, output and error, but we're not careful about how we use them. For the time being, most of these can just not be used.
  A. error
  B. warn
  C. info
  D. debug
  E. trace
2. A mechanism for getting samples back from Stan. Right now it's implemented through recorders that assume that it can do actions like push in a row of samples and write messages to a csv file. CmdStan just implements a way to stream to file. RStan implements a recorder that can 1. accept all samples or optionally accept a subset of samples and save that to memory and/or 2. stream to file.
3. A mechanism for getting diagnostic samples back from Stan. Right now, CmdStan streams that to file or ditches the diagnostic samples. RStan does the same. It might makes sense that this is combined with 2.
4. A callback for handling ctrl-c within interfaces. CmdStan does nothing. RStan implements a break using Rcpp, I think.

That's what I can think of so far. I'll try to come up with a design that captures what the interface needs to be.
