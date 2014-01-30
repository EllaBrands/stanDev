Andrew's Requirements for the Interface:

Outputs:

* Penalized mle on the transformed space 
* Covariance matrix on the transformed space
* Normal-based confidence intervals on the transformed space (by default, I guess these would be 95% intervals, even though I hate that), transformed back to the original space
* Simulations based on the normal approx, transformed pack to the original space (I don't know how many simulations, maybe 100 by default)
* (Maybe, if it works) estimated degrees of freedom for the multivariate t approximation; t based conf intervals on the transformed space, transformed back to the original space; simulations based on the t approx.

I can't think of anything more right now, although of course there could be something obvious that I'm missing.  I suppose we might also want to return various diagnostics of the fit, such as 

* the starting point, 
* the number of steps it took to converge, 
* the gradient at the last step

