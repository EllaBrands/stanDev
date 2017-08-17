### Roundtable
_Short (less than 1 minute) description of work in the past week._


### Unresolved Topics
_Address any open topics from the past meeting._

### New Topics
_New topics that should be addressed immediately or by the next
meeting._

* __Developer name.  Short description.  Desired resolution.__

Breck: Funding clearing house. 16 live proposals folks, we need some winnowing and help writing. 

Sebastian: Testing special code like MPI - how? Issues: (1) additional dependencies (MPI installation+build boost libraries), (2) requirement to build .cpp file with mpicxx compiler and (3) test executable must be started using "mpirun -np 4 mpi_test".

### Open Discussion Topics
_Any topics that do not need to be addressed in the short term,
including speculation and brainstorming._

* __Developer name.  Short description.__

Andrew.  (I won't be at the Stan meeting on Thurs and probably won't be able to join the hangout either, so others can discuss the topics below, or else we can put them off until when I return.)

Andrew.  Stan movies.  Stan blog.

Andrew.  Fitting discrete models in Stan (I put this on discourse:  http://discourse.mc-stan.org/t/fitting-markov-models-with-latent-discrete-states/1568 ).

Andrew.  Just checking on bayesplot paper for JRSS with Jonah and Andrew, as it's due end of Aug.

Bob.  Model concept base class access to variable declarations.  Current proposal here:  https://github.com/stan-dev/stan/issues/2370

Bob.  Managing project assets like URLs.  I don't like personally owning our URLs.  Can we transfer to NumFOCUS? 

Bob.  Probabilistic testing.  Existing tests test bytewise results rather than probabilistic results, so whenever anything changes in the density code or RNG code or we move to a new platform, we have to rethink everything as we're doing now w.r.t. 2.17.  Any ideas? 