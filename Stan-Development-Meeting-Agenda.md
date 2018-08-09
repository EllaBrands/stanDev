### Roundtable
_Short (less than 1 minute) description of work in the past week._

### Unresolved Topics
_Address any open topics from the past meeting._

### New Topics
_New topics that should be addressed immediately or by the next
meeting._

* __Date added. Developer name.  Short description.  Desired resolution.__

* 2018-08-09. Andrew.  Verbose options in stan, rstan, rstanarm.

* 2018-07-19. Andrew. Starting at our next meeting, we will try Sean's suggestion of first doing the Stan-adjacent research meeting (informally led by me) from 11 to 11:30, then at 11:30 starting the Stan core software meeting (informally led by Bob).  So people can join at either time, depending on their interest.  We'll see how that works out!

* 2018-07-27. Andrew.  Default priors for ordered logit.

### Open Discussion Topics

_Any topics that do not need to be addressed in the short term,
including speculation and brainstorming._

* __Date added. Developer name.  Short description.__


* 7 August 2018.  Bob.  Different integrators.  Ben Bales tells me that (a) he has a problem for which a hand-built Euler solver works but our built-in RK45 fails, and (b) that Yi is working on an adjoint sensitivity integrator that's going to still need the alloc-stack to malloc internally and get cleaned up.

Ben 2. Continuing on what Bob added, I'd also like to talk about adjoint sensitivity and the possibility of switching to using Cvodes sparse solves stuff with fvar Jacobian-vector products while we're on the topic of ODEs.

* 2018-07-12. Ben.  New Era of Correctness
```
// [[Rcpp::plugins(cpp14)]]
// [[Rcpp::depends(BH)]]
#include <Rcpp.h>
#include <boost/math/quadrature/tanh_sinh.hpp>
#include <boost/math/special_functions/pow.hpp>

// [[Rcpp::export]]
double log_besselK(const double v, const double z) {
  using std::exp;
  using std::fabs;
  using std::log;
  using std::pow;
  
  if (v == 0.5) return 0.5 * log(M_PI / (2 * z)) - z;
  const long double v_ = fabs(v);
  const long double v_mhalf = v_ - 0.5;
  const long double neg2v_m1 = -2 * v_ - 1;
  const long double lead = 0.5 * log(M_PI) - std::lgamma(v_ + 0.5) - v_ * log(2 * z) - z;
  if (isinf(lead)) return -z + 0.5 * log(0.5 * M_PI / z);
  const long double beta = 16.0 / (2 * v_ + 1);

  long double error;
  long double L1;
  size_t levels;
  long double condition_number;
  
  boost::math::quadrature::tanh_sinh<long double> integrator;
  long double termination = std::sqrt(std::numeric_limits<long double>::epsilon());

  long double Q = integrator.integrate([&](const long double u) {
    auto uB = pow(u, beta);
    auto first = beta * exp(-uB) * pow(2 * z + uB, v_mhalf) * boost::math::pow<7>(u);
    auto second = exp(-1.0 / u);
    if (second > 0) second *= pow(u, neg2v_m1) * pow(2 * z * u + 1, v_mhalf);
    return first + second;
  }, 0.0, 1.0, termination, &error, &L1, &levels);

  condition_number = L1 / fabs(Q);
  std::cout << "error = " << error << std::endl
            << "L1 = " << L1 << std::endl
            << "levels = " << levels << std::endl
            << "condition_number = " << condition_number << std::endl;
  
  return lead + log(Q);
}
```
