We can't really have a useful base class with template methods. The template parameters now are

* dropping constants (boolean `propto`)
    - true or false

* including Jacobian adjustment (boolean `jacobian`)
    - true or false

* RNG (class `BaseRNG`)

* scalar type for density evaluation (numbered by derivative order)

    - (0) `double`
    - (1) `var`
    - (2) `fvar<var>`
    - (3) `fvar<fvar<var>>`

Expanding that out would be 16 implementations.  `propto` can be controlled by the model to bring it down to 8.  Right now, Stan doesn't use the higher-order fvar, so that brings it down to 4:  +/- Jacobian, `double`, and `var`. 


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
