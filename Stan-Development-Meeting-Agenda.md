### Roundtable
_Short (less than 1 minute) description of work in the past week._

### Unresolved Topics
_Address any open topics from the past meeting._

### New Topics
_New topics that should be addressed immediately or by the next
meeting._

* __Developer name.  Short description.  Desired resolution.__
* Ben. RStan is broken on develop. Daniel said he would fix the writer fallout.

* Andrew. Fitting a model with latent jumps (from Novartis).  See graphs here:  http://www.stat.columbia.edu/~gelman/temp/Screen%20Shot%202016-03-07%20at%204.29.55%20PM.png


### Open Discussion Topics
_Any topics that do not need to be addressed in the short term,
including speculation and brainstorming._

* __Developer name.  Short description.__

* Ben. Can we do enough testing soon to support / move to C++11 for Stan3 [Link](https://github.com/stan-dev/stan/wiki/Cpp11-Upgrade)?

* Krzysztof. Is there interest in having a binary storage format for input data? output? non-data input?  Streaming read/write of data as key/value/index turns out to be pretty fast using [protobuf](https://github.com/sakrejda/protostan/blob/develop/src/protostan/interface_callbacks/writer/binary_proto_stream_writer.hpp) (e.g.- round-trip encoding/decoding time for 100,000 random doubles or short random strings using an interface_callbacks writer class was ~200ms), could be generally viable beyond CloudStan dreams(?) What are your preferred alternatives? [Notes](https://github.com/stan-dev/stan/wiki/Protocol-Buffers-for-serialization-of-input-data,-output-samples,-initial-values,-input-parameters,-and-output-messages,)

* Bob.  What is Stan 3 going to be and when are we going to release it?

* Bob.  How are we going to start the Stan Book?  Who's the audience going to be and what level(s) will the book be written with (Krushcke / Gelman & Hill / BDA or a mix)?  If we want to deploy on the web as PDF, which software to use to write it?  