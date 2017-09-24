
Not considering the issue of a virtual base class, I think we want something like this:

```
namespace stan {
namespace model {
class model {
model(const var_context& data);

~model();

long num_params();

template <bool propto, bool jacobian, typename T>
log_density(const std::vector<double>& unconstrained_params, 
            ostream& print_msgs) const;

template <class BaseRNG& rng>
vector<value> var_values(bool p, bool tp, bool gq,
                         const std::vector<double>& unconstrained_params,
                         std::ostream& print_msgs, BaseRNG& rng) const;

static std::vector<var_decl> var_decls(bool d, bool td, bool p, bool tp, bool gq);

std::vector<sized_var_decl> sized_var_decls(bool d, bool td, bool p, bool tp, bool gq) const;

void unconstrain(const var_context& c,
                 vector<double>& unconstrained_params) const;
};  // class model
}  // namespace model
}  // namespace stan

We can't really have a useful base class with template methods because template methods can't be declared as virtual (no way to allocate symbol table statically until instantiations are known).

The template parameters now are

#### Density function

* dropping constants (boolean `propto`)
    - true or false

* including Jacobian adjustment (boolean `jacobian`)
    - true or false

* scalar type for density evaluation (numbered by derivative order)

    - (0) `double`
    - (1) `var`
    - (2) `fvar<var>`
    - (3) `fvar<fvar<var>>`

#### Generated quantities

* RNG (class `BaseRNG`)

Expanding this all out would lead to 16 fully instantiated methods.

Can we let `propto` be controlled within the model itself?  Right now, there's an issue with `T = double, propto=true` in that all the `~` densities drop out.  Should we just let that go and leave it up to the model to use `target +=` to get the full density if they want it in `double` form?  Or do we need some kind of better external control where `propto` is really just "optimize for MCMC and don't calculate total density"?
