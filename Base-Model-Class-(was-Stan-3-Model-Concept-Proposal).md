#### Proposal for Model Concept

note:  description of existing model concept functions on
[this wiki page](https://github.com/stan-dev/stan/wiki/Model-Concept)

##### Design constraints/goals/etc:

1. model class can be compiled as a single translation unit

2. define full set of methods needed by interfaces for the full Stan workflow.

3. leverage modern C++


```
namespace stan {
namespace model {

// Class for holding data, transforming between constrained and
// unconstrained parameter values, and for evaluating log
// densities.
class model_base {

// Construct a model
// @param[in] var_context definitions of data variables
model_base(); 

// Destruct model.
virtual ~model_base();

// Read data from specified context
// @param[in] var_context definitions of data variables
void set_data(const var_context& data);

// Return number of unconstrained parameters.
// @return number of unconstrained parameters.
virtual long num_unconstrained_params() const;

// Return log density of unconstrained parameters, writing any
// program output to the specified output stream.
// @tparam propto true if constants dropped in sampling statements
// @tparam jacobian true if jacobian included for constraining transforms
// @tparam T scalar type
// @param[in] unconstrained_params sequence of unconstrained parameters
// @param[in,out] print_msgs stream for program print statements
// @return log density
template <bool propto, bool jacobian, typename T>
log_density(const std::vector<T>& unconstrained_params, 
  ostream& print_msgs) const;

// Return sequence of variable value objects in order of declaration.
// @tparam rng type of RNG used in generated quantities block
// @param[in] p include parameters
// @param[in] tp include transformed parameters
// @param[in] gq include generated quantities
// @param[in] unconstrained_params unconstrained parameter values
// @param[in,out] print_msgs stream to which print statements from program are written
// @param[in,out] rng base RNG for generated quantities
// @return sequence of constrained variable values
template <class BaseRNG& rng>
vector<value> var_values(bool p, bool tp, bool gq,
  const std::vector<double>& unconstrained_params,
  std::ostream& print_msgs, BaseRNG& rng) const;

// Return sequence of variable declarations in declaration order
// from specified blocks.
// @param[in] d include data
// @param[in] td include transformed data
// @param[in] p include parameters
// @param[in] tp include transformed parameters
// @param[in] gq include generated quantities
// @return variable declarations of specified blocks
static std::vector<var_decl> var_decls(bool d, bool td, bool p,
  bool tp, bool gq);

// Return variable declarations with sizes in declaration order
// from specified blocks
// @param[in] d include data
// @param[in] td include transformed data
// @param[in] p include parameters
// @param[in] tp include transformed parameters
// @param[in] gq include generated quantities
// @return sized variable declarations of specified blocks
std::vector<sized_var_decl> sized_var_decls(bool d, bool td,
  bool p, bool tp, bool gq) const;

// Set unconstrained parameters to values derived from
// the specified context.
// @param[in] c context defining constrained parameter values
// @param[in,out] unconstrained_params value set with unconstrained
// values
void unconstrain(const var_context& c,
  vector<double>& unconstrained_params) const;

};  // class model

struct var_decl {
  std::string type_;
  long num_dims_;
  bool has_lb_;
  bool has_ub_;
}

struct sized_var_decl : public var_decl {
  std::vector<long> dims_;  // constrained dimensions
}
  
struct value : public sized_var_decl {
  double lb_;  // -inf if unbounded below
  double ub_;  // +inf if unbounded above
  std::vector<double> value_;
};

}  // namespace model
}  // namespace stan
```

#### Base Class

We can't really have a useful base class with template methods because template methods can't be declared as virtual (no way to allocate symbol table statically until instantiations are known).

The template parameters now are

* (density function) dropping constants (boolean `propto`)
    - true or false

* (density function) including Jacobian adjustment (boolean `jacobian`)
    - true or false

* (density function) scalar type for density evaluation (numbered by derivative order)

    - (0) `double`
    - (1) `var`
    - (2) `fvar<var>`
    - (3) `fvar<fvar<var>>`

* (generated quantities) RNG (class `BaseRNG`)

Expanding this all out would lead to 16 fully instantiated methods.  Can we let `propto` be controlled within the model itself?  Right now, there's an issue with `T = double, propto=true` in that all the `~` densities drop out.  Should we just let that go and leave it up to the model to use `target +=` to get the full density if they want it in `double` form?  Or do we need some kind of better external control where `propto` is really just "optimize for MCMC and don't calculate total density"?

DanSimpson: It might be nice to have this depend on a flag. Usually, the current default is exactly what you want, but if someone wants to do something with marginal likelihoods for whatever reason, it would be much cleaner to have an interface parameter like "normalized = TRUE" that would tell `~` to set `propto=false`.  Bob:  That control already exists in the model concept with template parameter `propto`.  (The interfaces are a separate issue---it would be relatively easy to add this feature to `stan-dev/stan` in `stan/services`.)

#### Iterators?

An alternative to the `std::vector<T>` type for unconstrained parameters, we could have constant or writeable iterator begin or begin/end pairs.