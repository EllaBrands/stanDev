This page serves to collect ideas for an official Stan Docker image. In a first step the use cases and requirements should be defined from which the technical implementation should be derived from.

# Use cases

- Easy deployable (Docker)!
- Debugging and development of Stan by developers
- Reference platform for running Stan models in a reproducible manner (maybe use some Ubuntu LTS as basis?)
- Persistency / interaction with host system
- Harmonize interfaces (compiler & compiler settings)
- Rapid availability with each release (maybe even a dev version supporting to create the image from git hashes)
- Integration with Jenkins?
- RStan image should integrate the base Stan docker image and the Rocker project ideally, see https://hub.docker.com/u/rocker/

# Features

- fast compiles (clang?)
- different compiler settings for modelling or Stan development
- support for compilation with ccache to speedup re-compilation 
- persistency through Docker volumes(?) or local mounts
- pre-deploy Stan libraries such as the TBB, boost?