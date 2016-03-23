# Requirements suggested so far, with options
- easily plugged into libraries that require .csv input
 - compatible with R's data.table::fread (no comment lines)
 - compatible with... 
- streaming
- container for multiple data types (samples, diagnostic, etc...)
 - directories
 - .zip/.tar.gz
 - protobuf
 - jsonlines
- No comment characters intermixed with draws; any comment characters should be in a separate file or failing that at the top or bottom of the file. See issue [1321](https://github.com/stan-dev/stan/issues/1321)

# Exploration

## protobuf
 - keeps binary representation of doubles for restarting sampling.
 - would require an easy path (utility function + separate executable) for producing a set of .csv files.
 - writing a stream of delimited protobuf messages is part of the Java protobuf API, and should soon be part of the C++ protobuf API.  We have an [implementation](https://github.com/sakrejda/protostan/blob/develop/src/protostan/util/rw_delimited_pb.hpp) compatible with both of those.  
 - the current protobuf [writer](https://github.com/sakrejda/protostan/blob/develop/src/protostan/interface_callbacks/writer/binary_proto_stream_writer.hpp) turns everything (matrices and vectors) into key/value format, it's still fast (but **NOTE: add comparison to .csv writer**)
 - if we wanted the writer to be faster we could still optimize by writing chunks rather than key/value.

## multiple .csv files wrapped in .zip or .tar.gz
 - Re:streaming multiple files: looks like writing to [multiple files](http://unix.stackexchange.com/questions/13093/add-update-a-file-to-an-existing-tar-gz-archive) at once would need to happen prior to tar/zip step.  
 - There is a library that appears to be carefully maintained, [multi-platform](http://www.libarchive.org), and would ease the whole directory/disk [manipulation](https://github.com/libarchive/libarchive/wiki/Examples#constructing-objects-on-disk) thing.
 - The library license says New BSD on the sticker but is currently a [mess](https://raw.githubusercontent.com/libarchive/libarchive/master/COPYING), sigh...
 - Other library suggestions?

## jsonlines
 - One line per JSON-format "message"
 - http://jsonlines.org
 - could be easily produced from protobuf format
 - would still need to be parsed to produce .csv output.
