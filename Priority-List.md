* (Daniel) Refactor Stan backend to make RStan, CmdStan, PyStan easier to code and maintain

* Separating parameters into bins

* (Andrew, Ben, Jiqiang) RStan install 

* (Michael + support) new 1st, 2nd, 3rd derivatives for everything

    * includes RHMC

*  2nd derivs for current Stan

    * finite diff gradients in Stan
    * finite diff gradients in R
    * existing functions (w/o probabilities, etc.)

* (Bob + support) adding functions to Stan (3--4 weeks)

* adding differential equation models (depends on 2):  

    * requires adding functions
    * efficient auto-diff version of derivs
    * requires higher-order derivatives for stiff systems

* (Andrew + Jiqiang + Ben) adding to RStan

    * computing WAIC in RStan (eventually move to Stan)
    * CODA output
    * install instructions and scripts
    * saving fitted models across sessions
    * Andrew's list of desiderata (e.g., compile and run; makelike behavior for building model)

* (anyone) parameter blacklist (exclude) <b>or</b> whitelist (include)
       
* (Bob or Daniel) adding a mixture distribution

    * syntax design, either as something explicit or as something vectorized (use functions)?

        ```
        y ~ mixture(lambda, normal(mu[1],sigma[1]), ..., normal(mu[K],sigma[K]));
        ```
        ```
        y ~ mixture(lambda, normal, mu, sigma);  // vectorize
        ```

    * implementation and testing

* mle and penalized mle

* marginal mle and penalized marginal mle

    * requires partitioning into two groups, one of which is marginalized over (maybe general labeling)
    * optimize group 1, group 2 held constant
    * second derivative matrix conditional on parameters in group 2
    * all brute force, not factoring the model

* ep-like algorithms 

* computing normalizing constants

    * path sampling (e.g., temperature parameters), requires pulling out a single parameter

* black-box vb:  requires partitioning of parameters

* partition parameters based on name