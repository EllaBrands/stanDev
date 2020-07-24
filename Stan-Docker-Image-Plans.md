This page serves to collect ideas for an official Stan Docker image. 

# Use-cases
- Self-contained environment for teaching
- Reference stack to build upon for doing reproducible research
- Debugging and development of Stan by developers
- Automated build and test system

The question is how exactly the group satisfies a particular use case when the `Stan` project offers so many options to potential users. It might make more sense to think about how technical implementations map onto use-cases, with the hope to minimize the number of different images we would need to maintain.

- An interactive session like `JupyterHub` or `Rstudio server` would be invaluable in a class but unnecessary when trying to do integration testing.
- Applied researchers using one of the R interfaces would likely want a number of other `Stan` connected R packages, while those adding functionality to `Stan Math` probably don't need `posterior` or `loo`.

## Important Questions
- Which interfaces and components in StanDev should get their own dedicated image?
- Should pre-compiled be included?
- Which existing Docker software stack(s) should these containers build upon?

# Jupyter Docker Stack Proposal


# Desired Features
- fast compiles (clang?)
- different compiler settings for modelling or Stan development
- support for compilation with ccache to speedup re-compilation 
- persistency through Docker volumes(?) or local mounts
- pre-deploy Stan libraries such as the TBB, boost
- MPI enable / disabled
- Integration with Jenkins
- Integration with Rocker project
- Integration with JupyterHub
- Rapid availability with each release (maybe even a dev version supporting to create the image from git hashes)
