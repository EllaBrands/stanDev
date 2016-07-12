<!--
A functional specification describes how a product will work entirely from the user's perspective. It doesn't care how the thing is implemented. It talks about features. It specifies screens, menus, dialogs, and so on. 

1.    To let the developers know what to build.
2.    To let the testers know what tests to run.
3.    To let stakeholders know what they are getting.
-->

# Goals

The main goal is to get outsiders interested in and aware of Stan.

We want a demonstration system so that a potential user can see Stan running in real time and can play with the model.

Key features:
- Can be run from browser (so that the user does not need to download R or Stan).
- User should be able to change the Stan program and re-run.
- User should be able to change the data and re-run.
- There should be a library of examples (example includes Stan program, data, and a paragraph of background explanation).
- It would be good if the user could see the chains moving while Stan is running.  Simplest approach would be for user to pick two parameters and then there could be a graph showing the 4 chains in 4 different colors, showing the trace of the two parameters.
- Also it would be cool to display the steps of individual HMC iterations.

It's ok if it requires a password.  So that way we can use our own server and not worry about people overwhelming it.  If CloudStan works well, we can thinking about getting funding to host it on a server so anyone can use it, or maybe have a system where people pay for the server time.  But no need to worry about this right away.

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

# Mostly Complete Free Implementation

1. Create an account at https://cloud.sagemath.com . You can use your Google, GitHub, etc. account.
2. After signing in, create a "Project" called "test" or something. You do not need to upgrade.
3. Go to this [link](https://cloud.sagemath.com/projects/9cead8b7-e246-4ed2-b46f-9ad93dff442f)
4. Check the hyperlinked word StanMathCloud
5. Click the icon called "Check all" to mark the 3 files
6. Click the icon called "Copy"
7. In the dropdown menu for "In the project", select "test" or whatever you called your project
8. Click the icon called "Copy 3 items"
9. Click on the "Projects" item in the top left of the browser window
10. Click on the hyperlinked word "test" or whatever you called your project
11. Click on the hyperlinked word "StanMathCloud.ipynb"
12. You can edit any cell in this Jupyter Notebook and execute the cell by pressing Control+Enter. Or you can execute all cells by clicking Cell->Run All

If we had a [paid](https://cloud.sagemath.com/policies/pricing.html) plan, we could access extra features, such as internet connectivity. 

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

Each of these are already included on SageMathCloud.

# Flowchart (screen by screen specification)


## Home Page

What should this look like?

## Log In Form

None for now.


## Editor pane

## Run pane

## Results pane

## Download / export?





