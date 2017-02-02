One of the exciting items on the horizon for Stan is incorporating more and more Gaussian process covariance functions, especially those that exploit structure for faster computations.  At the same time, that structure will be cumbersome to propagate through the various computations needed to incorporate a Gaussian process in Stan, including determinant calculations and matrix divisions.  To ensure that Gaussian processes in Stan can take advantage of as much intrinsic structure as possible without requiring a completely suite of specialized linear algebra we have decided that each specialized Gaussian process covariance function introduced in Stan must have three implementations.

## Dense Covariance Matrix

Each Gaussian process implementation must have a function that returns a dense covariance matrix,

    matrix gp_name_cov(...)

This dense matrix will lose any structure inherent to the specific Gaussian process, but ensure that it can always be incorporated into composite Gaussian processes that combine different covariance functions.

## Cholesky Factor

Additionally, each implementation must have a function that returns a Cholesky factor of the covariance matrix,

    matrix gp_name_chol(...)

from which we can implement a non-centered parameterization of the Gaussian process.

If the covariance function is structured then that structure can be exploited in the C++ implementation of this function to realize the corresponding speed up.  

## Log Probability Density Function

Finally, each implementation must have a function that returns the log probability density function of a zero-mean multivariate normal with the Gaussian process's covariance matrix,

    real gp_name_lpdf(...)

from which we can implement a centered parameterization of the Gaussian process.

Again, if the covariance function is structured then that structure can be exploited in the C++ implementation of the log density function, in particular the calculation of the log determinant and precision quadratic form.  If there is no structure to exploit then this will reduce to the ```multi_normal_lpdf``` function, which can be called directly to avoid code duplication.
