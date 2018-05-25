## Construction

On [discourse](http://discourse.mc-stan.org/t/proposal-for-consolidated-output/4263/9?u=sakrejda) Bob suggests that we make it possible to create a var_context using a builder pattern:

> For all of this to be super flexible, it’d also be nice to have a var_context_builder, something like:
> `var_context vc = var_context::builder().matrix("a", m).real("f".2.3);`
> Not quite sure how to implement that, though.

## Suggested changes

1. Move dim validation from model to var_context
2. Remove validation from model
3. Add unconstrained to constrained transform map
4. Change parser

New: have the model pass transformation functors
to the var_context

Move transforms into their own classes: http://discourse.mc-stan.org/t/var-context-what-do-we-want-from-it/3665


## Code examples

These are from @betanalpha:

```
// DATA ACCESS LAYERS

// base_dal
vector_t get_vector(const std::string& name, size_t N) {
  if (!contains_r(name)) {
    std::stringstream msg;
    msg << "Variable " << name
        << " not found in variable context" << std::endl;
    throw std::runtime_error(msg.str());
  }

  std::vector<size_t> dims = dims_r(name);
  if (dims.size() != 1) {
    std::stringstream msg;
    msg << "Requested the vector " << name
        << ", but " << name
        << " is not defined as a vector in variable context"
        << std::endl;
    throw std::runtime_error(msg.str());
  }

  if (dims[0] != N) {
    std::stringstream msg;
    msg << "Requested the vector " << name
        << " with size " << N << ", but " << name
        << " is defined with size " << dims[0] << " in variable context"
        << std::endl;
    throw std::runtime_error(msg.str());
  }

  std::vector<double> vals = vals_r(name);
  vector_t output(N);
  for (vector_t_idx n = 0; n < N; ++n)
    output(n) = vals[n];
  return output;
}

// constr_dal
template <typename T>
void get_constrained_vector(const string& name,
                            const stan::io::transform& transform,
                            vector_t param) {
  param.resize(transform.size());
  param = get_vector(name, transform.size());
  transform.validate(param);
}

// unconstr_dal
template <typename T>
void get_unconstrained_vector(const string& name,
                              const stan::io::transform& transform,
                              vector_t::InnerIterator& it) {
  vector_t constr_val = get_vector(name, transform.size());
  transform.unconstrain(constr_val, it);
}

// EXAMPLE TRANSFORM
template <typename T, typename T_LB>
class vector_lower_bound: public vector_transform<T> {
private:
  T_LB lower_bound_;

public:
  vector_lower_bound(T_LB lower_bound, size_t N):
    lower_bound_(lower_bound), vector_transform<T>(N) {}

  size_t unconstrained_dim() { return N_; }

  void validate(vector_t constr_val) {
    stan::math::check_greater_or_equal("stan::io::vector_lower_bound",
                                       "Lower bound constraint", constr_val,
                                       lower_bound_);
  }

  template <typename Jacobian>
  vector_t constrain(const vector_t& unconstr_val, lp_accumulator<T_LB>& acc) {
    vector_t constr_val;
    for (idx_t n = 0; n < N_; ++n) {
      if (Jacobian) {
        T_LB lp;
        output(n) = stan::prob::lb_constrain(unconstr_val[n], lower_bound_, lp);
        acc.push_back(lp);
      } else {
        output(n) = stan::prob::lb_constrain(unconstr_val[n], lower_bound);
      }
    }
  }

  vector_t constrain(const vector_t& unconstr_val,
                     std::vector<double> constr_val) {
    for (idx_t n = 0; n < N_; ++n)
      constr_val.push_back(stan::prob::lb_constrain(unconstr_val[n],
                                                    lower_bound_));
  }

  void unconstrain(const vector_t& constr_val, vector_t::InnerIterator& it) {
    for (idx_t n = 0; n < N_; ++n)
      *(it++) = stan::prob::lb_free(constr_val[n], lower_bound_);
  }
};

// MODEL USE CASES

// constructor
model_name {
  ...
  vector_lb_transform<input_type, lb_type>
    param_name1_transform__(lb_value);
  data_var_context.get_constrained_vector("param_name1",
                                          param_name1_transform,
                                          param_name1__);
  ...
}

// init
void init(vector_t& unconstr_state) {
  vector_t::InnerIterator it(unconstr_state);

  ...
  vector_lb_transform<input_type, lb_type>
    param_name1_transform__(lb_value);
  get_unconstrained_vector("name1", param_name1_transform, it);
  ...
}

// what write_array was
void constr_state(vector_t unconstr_state,
                  std::vector<double>& constr_state) {
  ...
  vector_lb_transform<input_type, lb_type>
    param_name1_transform__(lb_value);
  param_name1_transform__.constrain(unconstr_state, constr_state);
  ...
}

// log_prob
template <typename T>
T log_prob(vector_t& unconstr_state) {
  lp_accumulator<T> lp_acc__;

  ...
  vector_lb_transform<input_type, lb_type>
    param_name1_transform__(lb_value);
  vector_t param_name1__ =
    param_name1_transform__.constrain<Jacobian>(unconstr_state, lp_acc__);
  ...
}
```


```
Current:

model::initialize(var_context) {
  for parameter in parameters {
    var_context::does_parameter_exist(parameter);
    values = var_context::get_values(parameter):
    var_context::verify_dimensions(parameter, dims);
    internal_parameters.append(map_to_unconstrained(values));
  }
}

1. Move dim validation from model to var_context
2. Remove validation from model
3. Add unconstrained to constrained transform map
4. Change parser

New: have the model pass transformation functors
to the var_context,

model::initialize(var_context) {
  for parameter in parameters {
    var_context.get_unconstrained<transform>(internal_parameters, parameter_name, constrained_dim);
  }
}

where

template <Transform>
void var_context::get_unconstrained(const vector<double>& parameters,
                                    const string& parameter_name,
                                    int constrained_dim) {
  if(   does_parameter_exist(parameter_name)
     && verify_dimensions(parameter_name, constrained_dim))
    parameters.push_back(Transform::unconstrain(get(parameter)));
  else
    parameters.push_back(andom_array(Transform::unconstrained_dim()));
  
  std::vector<double> get_values(std::string parameter_name);
  bool does_parameter_exist(std::string parameter_name);
  bool verify_dimensions(std::string parameter_name, int constrained_dim);
}

and a transform functor looks like

class transform {
  static virtual void constrain(const vector<double>& input, vector<double>& output) = 0;
  static virtual void unconstrain(const vector<double>& input, vector<double>& output) = 0;
  static int constrained_dim();
  static int unconstrained_dim();
}

<double LowerBound>
class lb_transform: public transform {
  static void constrain(const vector<double>& input, vector<double>& output) {
    output.push_back(stan::prob::lb_constrain(input, LowerBound));
  }
  static void unconstrain(const vector<double>& input, vector<double>& output) {
    output.push_back(stan::prob::lb_free(input, LowerBound));
  }
  static int constrained_dim() { return 1; }
  static int unconstrained_dim() { return 1; }
}


var_context() {
public:
  bool check_r(string name, vector<int> dims);
  vector<double> get_r(string name, vector<int> dims);
}

template <typename RNG>
rand_var_context(): public var_context {
public:
  rand_var_context(RNG& rng);
  bool check_r(string name, vector<int> dims);
  vector<double> get_r(string name, vector<int> dims);
private:
  RNG& rng_;
}

template<typename PRIMARY_VC, typename FALLBACK_VC>
fallback_var_context {
  fallback_var_context(PRIMARY_VC& primary, FALLBACK_VC& fallback,
                       vector<string> names, vector<vector<int>> dims) {
    
    
    
  }
  bool check_r(string name, vector<int> dims);
  vector<double> get_r(string name, vector<int> dims);
  bool is_fallback_utilized();
private:
  PRIMARY_VC& primary_;
  FALLBACK_VC& fallback_;
}

template<typename M,
         typename INIT_VC,
         typename RNG>
void initialize_state(M& model, INIT_VC& init, RNG& rng) {
  
  typename rand_var_context<RNG> rand_vc;
  rand_vc rand(rng);
  fallback_var_context<INIT_VC, rand_vc> rand_fallback(init, rand,
                                                       model.names(), model.dims());
  
  if (rand_fallback.is_fallback_utilized()) {
    RECORDER << "Hey, I'm using a partial init" << endl;
    for (int n = 0; n < 100; ++n) {
      model.transform_inits(rand_fallback, cont_params);
      // check log_prob
      // check_grad_log_prob
    }
  } else {
    model.transform_inits(rand_fallback, cont_params);
    // check log_prob
    // check_grad_log_prob
  }
  
}
```