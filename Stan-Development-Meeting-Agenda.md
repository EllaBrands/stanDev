### Roundtable
_Short (less than 1 minute) description of work in the past week._

### Unresolved Topics
_Address any open topics from the past meeting._

### New Topics
_New topics that should be addressed immediately or by the next
meeting._
* __Date added. Developer name.  Short description.  Desired resolution.__

* 25th April. Sebastian. Is it OK to use `mingw32-make` on Windows to build Stan-math (and all other `cmdstan` depending interfaces)? This is right now needed to build the Intel TBB on Windows (we still need to figure out why the test-runs take much longer right now with the TBB just being linked in). Desired resolution: It's fine, but that means to change some doc in the project about build instructions and impacts other interfaces.

### Open Discussion Topics

_Any topics that do not need to be addressed in the short term,
including speculation and brainstorming._

* __Date added. Developer name.  Short description.__

18 Apr 2019. Sean. What happens if you try to fit complete noise with logistic regression?

23 Apr 2019. Andrew.  Static analysis so that Stan does not need to re-evaluate the autodiff tree at each step.  Aki says this could be huge for GP's.