### Roundtable
_Short (less than 1 minute) description of work in the past week._


### Unresolved Topics
_Address any open topics from the past meeting._

### New Topics
_New topics that should be addressed immediately or by the next
meeting._

* __Developer name.  Short description.  Desired resolution.__
 
* Sebastian. MPI Stan functions. Do we need any more tests of the MPI prototype or can we move forward with the signatures proposed (see [here](http://discourse.mc-stan.org/t/mpi-design-discussion/1103/233)). A short discussion of the input format of the parameters would be great; I am happy to explain concerns about it (Stan language consistency / performance considerations).

* Sebastian. MPI test system. I figured how to run MPI tests, but this requires to introduce a custom `main` function for tests. Will this suffice and can we adapt the makefile system? See [here](https://github.com/stan-dev/math/blob/d717be647beb06e5c5419caaf83ff923ae02331b/test/unit/math/prim/mat/functor/map_rect_mpi_test.cpp#L94).

* Matthijs. Master's projects. I'd like to advertise two possible thesis projects for MSc students, here in Oxford: one on the reliability of VI and one on compiler optimisations for probabilistic programming. The idea is that both would involve using and hopefully contributing to Stan. I'd like to see if anyone has any suggestions / comments.

### Open Discussion Topics
_Any topics that do not need to be addressed in the short term,
including speculation and brainstorming._

* __Developer name.  Short description.__


Andrew.  We can briefly review all the papers/projects we're working on with Dan, Aki, etc.  I think there are about 10 of them (GMO, hlm's, weak priors, simulation-based checking, Bayesian workflow, a few more I can't remember right now).

Andrew.  Stan demo and movie.

Andrew.  Stan manual:  add change-point models.

Andrew.  Stan modeling cheat sheet.

Andrew.  Stan intro material.