### Roundtable
_Short (less than 1 minute) description of work in the past week._


### Unresolved Topics
_Address any open topics from the past meeting._

### New Topics
_New topics that should be addressed immediately or by the next
meeting._

* __Developer name.  Short description.  Desired resolution.__

Breck. Scut Week--5 days of 'some day' for all the annoying tests, documentation or cruft that we have been saying we will get to 'some day'. 

Charles. Tests on Laplace approximation. When the latent Gaussian is "high-dimensional" (M = 100 or 500), what kind of solver should we use? Powell is slow and Newton unstable. What about gradient descent?

Sebastian. Is there interest in a super-fast singleton in stan-math ([the one from here](https://www.theimpossiblecode.com/blog/c11-generic-singleton-pattern/))? Yes/No. If yes, where to stuff it? Warning: this must use pointers (but I think that's really OK).

Old: Bob Carpenter.  Can we add a bunch new devs who've been adding things but aren't on our dev list?

Old: Ben Goodrich. What is up with numerical integration? Should we just use Boost's?

### Open Discussion Topics

_Any topics that do not need to be addressed in the short term,
including speculation and brainstorming._

* __Developer name.  Short description.__

Andrew.  Various user issues
- Running Stan for fixed amount of time instead of fixed number of iterations
- other issues