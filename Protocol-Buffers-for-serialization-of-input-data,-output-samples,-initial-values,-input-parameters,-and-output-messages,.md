## Proposal - In Progress, summarizing dev-list discussion...

Create a small library with Stan/Math as submodules that allows an interface to use Protocol Buffers for data input files to Stan, output of samples, input of initial values, input of control parameters, and output of messages.  The best use case is that it reduces the complexity of a client-server version of Stan dramatically.  

##  Protocol Buffers

Protocol Buffers is intended for (cross-platform)[https://developers.google.com/protocol-buffers/docs/overview#what-are-protocol-buffers] sharing of (small messages)[https://developers.google.com/protocol-buffers/docs/techniques#large-data].  The format is binary and described externally in .proto files.  An example .proto file is:

~~~
message SearchRequest {
  string query = 1;
  int32 page_number = 2;  // Which page number do we want?
  int32 result_per_page = 3;  // Number of results to return per page.
}
~~~

The protoc compiler takes this .proto file and uses plugins to generate code for any (supported)[https://developers.google.com/protocol-buffers/docs/reference/overview] languages.  The result is native code which can construct/read well defined messages.  The Protocol Buffers language definition specifies data types, with (mapping)[https://developers.google.com/protocol-buffers/docs/proto3#scalar] to data types in supported languages.  

## Benefits

* Binary floating point numbers so round-trip possible.
* Library takes care of message versioning, binary storage format.
* protoc compiler generates most of the code for interfaces to use the format.
* Google-size commitment to inter-operability among languages---that's what we (should) want.

## Pitfalls

* Potentially more complicated than a bespoke Stan serialization format.
 - Our data types are simple and relatively standard (Eigen types, or basic C/C++ types).
 - I (Krzysztof) don't want to think about versioning and other serialization issues that have already been solved by boost::serialize/protobuf/BSON/etc...
* Messages should be small, with a suggested "reconsider your strategy" limit of (1MB)[https://developers.google.com/protocol-buffers/docs/techniques#large-data].
 - Message size limits suggest that larger vectors/arrays/matrices need to be chunked for storage/transmission    
   if this is used as a generic format.
 - If chunking is complicated, we loose the benefit of handing message serialization/construction off to a 
   library.  An interface should be able to call one function and serialize a vector, not worry about chunking 
   it so the serialization format is satisfied.  
* The library (except for the Java plugin/version...) does not deal with writing multiple delimited messages to a stream/file. There is a (standard)[http://stackoverflow.com/questions/2340730/are-there-c-equivalents-for-the-protocol-buffers-delimited-i-o-functions-in-ja] (outside of Google) way of doing this suggested by one of the original Protocol Buffers devs.  
 - There is an outstanding Protocol Buffers (pull request)[https://github.com/google/protobuf/pull/710] by the same dev that would add a standard format for serializing multiple messages for streaming to a file/elsewhere.  This would solve part of the problem as interfaces would auto-generate the code for dealing with sending multiple messages in a single stream.
 - Otherwise a few well-tested C/C++ functions could take care of the issue for all the interfaces.
