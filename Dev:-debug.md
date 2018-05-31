The following only applies to Math library.

There is minimalist debug build, replacing `-O3` with `-g -O0` in `CXXFLAGS`.

To switch on debug build, add the following line in `make/local`
```bash
DEBUG = 1     # debug build
```
In addition to the usage of debuggers such as GDB, debug build turns on DEBUG macros, so in C++ code one can do
```c++
#ifdef DEBUG
// ...
#endif
```
Last but not least, `cout` and `typeid` is your friend.