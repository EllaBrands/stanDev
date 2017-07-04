### Goal

The goal is to be able to run a Stan program for previously drawn parameters running only the generated quantities block.  This will allow a sample (a number of draws of parameters from the posterior) to be used to make predictions or estimate event probabilities within Stan.  Presently, the generated quantities have to be defined and calculated at the same time the model is fit.  Errors in generated quantities calculations can derail sampling, so it will additionally be safer.

### Basic Functional Specification

* Develop an interface command (like the existing sampling, optimization, and diagnostic commands) for running a model's generated quantities block on previously generated draws of parameters
    * read in previous draws of parameters on constrained scale as matrix [2--3 weeks (requires model tweaks)]
    * read in previous data (it may be needed in the generated quantities calculation, e.g., posterior predictive checks) [1 day]
    * output generated quantities as callbacks in the same way as parameters [1 week]
    * handle print statements in program and error messages on callbacks [2 days]
    * validate input so that all parameters required by program exist on input [1 week]

* Develop input method for vector of constrained parameters paralleling the var_context input that currently exists [1--3 weeks as part of larger effort to refactor I/O to protobufs or JSON;  1--2 weeks as another I/O hack]

* Develop CmdStan wrapper.  Write manual chapter on how it is used [1 week]
    * This will require generalizing the way the current CSV reader works and that code's a mess, so it will have to be refactored [1--3 weeks]
    * It will also involve generating new upstream tests for math and stan libs [1--2 days]

* Develop RStan wrapper.  This will work on a fit object and a new Stan program.  Write vignette on how it's used. Xplatform test for CRAN. This must involve RStan developers [1--2 weeks]

* Develop PyStan wrapper.  Write documentation for it online. This must involve PyStan developers [2--3 weeks] 

All development is assumed to entail design, code documentation, code, unit tests, manual documentation, and code review.

All in, this looks like about 4 person months and will need to involve a distributed team:

* C++: language level code generation for transforms in stan
* C++: services callback wrapper in stan
* C++: low-level I/O for cmdstan
* R: for rstan
* Python: for pystan