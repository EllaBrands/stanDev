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

# SOME OLDER STUFF FROM PREVIOUS PAGE

The lmer emulator (or, really, the lmer/glmer/blmer/bglmer emulator)

Partition the parameters theta into global parameters phi and local parameters alpha.  This will be an algorithm to find the marginal posterior mode of phi, that is, the value of phi that maximizes 

p(phi | y) = INTEGRAL p(phi,alpha | y) d.alpha.  

For a hierarchical linear model ("lmer" or "blmer"), the algorithm will give the exact posterior mode, making use of the fact that the conditional posterior distribution p(alpha | phi,y) is normal.  For a nonlinear model ("glmer" or "bglmer"), it should give an approximate mode based on a conditional Laplace approximation to p(alpha | phi,y).

The algorithm goes as follows.  There is an outer loop that optimizes over phi, and an inner loop that integrates over alpha.
The outer loop goes as follows:

* (1) Start with a guess of phi.

* (2) Then do the inner loop:

    * (a) Start with a guess of alpha.

    *  (b) Run the optimizer on p(phi,alpha | y) with phi held fixed.  Thus, find the alpha that maximizes p(phi,alpha | y) conditional on the chosen value of phi.  This is equivalent to finding the conditional posterior mode of p(alpha | phi,y).  This optimizer can (should) use gradient information.  The only trick is that it is all conditional on phi, so it's optimizing only over the alpha-space.

    *  (c) Compute the 2nd-derivative matrix of log p(phi,alpha | y), computing derivatives only for the alpha dimensions.  This is equivalent to computing the 2nd-derivative matrix of log p(alpha | phi,y).

    *  (d) Approximate integral p(phi,alpha|y) d alpha (conditional on the current value of phi) by computing the value of p(phi,alpha | y) at the mode computed in step b, along with the determinant of the matrix from step c.  For a normal linear model (the "lmer/glmer" case), this will be the exact value of the integral.  For a nonlinear model (the "glmer/bglmer" case), it will be an approximation.

* (3) Now try a new value of phi.  Then repeat step 2.

* (4) Keep iterating steps 2 and 3 so that p(phi | y) (or, more precisely, the approximation computed in step 2d) increases.  Stop at the (approximate) mode.

The only trouble here is that it doesn't look to me that step 3 can be done using gradients.  It looks like the outer loop has to be done as a crude, non-gradient optimizer.  But I think that's what lmer/glmer does, and that works ok.  So, if I'm not confused here, the above algorithm would be a "lmer emulator" that, I hope, would be faster and more stable than lmer/glmer, as it would not require least-squares solutions in the inner loop.

Betancourt added via e-mail:   Actually, using tricks like those used in the SoftAbs calculations, you should be able to get the gradient of p(phi, alpha | y), too, to assist in 3 if we _really_ wanted to push lmer but I think our efforts are better served in VB and such.
