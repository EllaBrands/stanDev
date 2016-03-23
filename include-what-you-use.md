# Installation on Mac
Easy installation requires Homebrew.

Steps:

1. `brew tap jasonmp85/iwyu`
2. `brew install iwyu`

More details: https://github.com/jasonmp85/homebrew-iwyu


# Running include-what-you-use on the parser
Once iwyu has been installed:
```
make -k test/test-models/stanc CC=iwyu &> iwyu.out
```

Inspect `iwyu.out`.

More info:
- `-k` flag continues running make in the presence of errors. iwyu always returns an error code, so omitting that will stop at the first compilation.
- `CC=iwyu` just replaces the compiler with iwyu. iwyu doesn't support the full set of compiler flags, so it'll put up warnings about some of our compiler flags.
- `&> iwyu.out` just grabs the output in both the stdout and stderr and puts it into iwyu.out.


# Running include-what-you-use on the rest of the code base

I haven't figured it out yet.