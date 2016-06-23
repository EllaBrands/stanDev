<!--
A functional specification describes how a product will work entirely from the user's perspective. It doesn't care how the thing is implemented. It talks about features. It specifies screens, menus, dialogs, and so on. 

1.    To let the developers know what to build.
2.    To let the testers know what tests to run.
3.    To let stakeholders know what they are getting.
-->


# Overview

CloudStan is a client for Stan that runs from a browser. It will allow users to:

- connect to a remote server (where computation will be run)
- load, edit, and save Stan programs on that server
- instantiate a compiled Stan program with data that is either on the server or local
- run Stan
- track progress on the Stan program
- show output; plot
- ability to download results


## Major Features

- Run Stan without local installation
- No interface-specific code
- Stan program editor; must have syntax highlighting
- Library of example Stan programs
- Run Stan program
- Data: load on server / upload; basic exploratory cabability
- Multiple panes / tabs
- Syntax checker


# Scenarios

It helps to think through some scenarios and how these users would use it.

## Scenario 1: Andrew

Wants to:

- load an existing Stan program
- change existing programs
- run it and walk away
- come back and see the results


## Scenario 2: Complete Stats Novice


Wants to:

- throw their dataset at a number of statistical models
- evaluate which are best


## Scenario 3: New Stan User

Wants to:

- Get started with a Stan program
- See what steps are happening:
    * Translation from Stan to C++
    * C++ to machine code
    * Iterations
    * Possibly know some within-iteration information
- Run a different Stan program


# Non Goals

CloudStan will not support the following features (to start):

- collaboration
- multiple concurrent sessions
- version control
- run multiple programs at once
- history of runs


# Flowchart (screen by screen specification)


## Home Page

What should this look like?

## Log In Form

None for now.


## Editor pane

## Run pane

## Results pane

## Download / export?





