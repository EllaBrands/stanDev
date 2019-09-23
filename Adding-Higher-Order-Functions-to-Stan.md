Update: This procedure refers to the old compiler, Stan 2. We're attempting to move all compiler work to [stanc3](https://github.com/stan-dev/stanc3). Adding a higher order function is described in [this youtube video](https://www.youtube.com/watch?v=mvLa1LIQvBA) and [this exemplar PR](https://github.com/stan-dev/stanc3/pull/303) (it's much simpler now, thankfully!).

# Introduction
Higher-order functions are functions that take other functions as arguments.  An example of such a function is an Ordinary Differential Equation integrator. `Add()` doesn’t handle arguments which are functions, so we need to do more work to expose higher-order functions to Stan.

Instead of editing `function_signatures.h`, we directly modify Stan’s grammar. This can be an involved process, but has been done previously for other higher-order functions like `integrate_ode_rk45()`, and `algebraic_solver()`. The steps to add a higher-order function include editing:

- The Stan grammar
- The abstract syntax tree (AST)
- The expression generator
- The AST visitors
- The `stan::math` library

This discussion assumes some familiarity with the easier process of adding new (normal) functions to Stan as outlined [here](https://github.com/stan-dev/math/wiki/Adding-a-new-function-with-known-gradients) and [here](https://github.com/stan-dev/stan/wiki/Contributing-New-Functions-to-Stan). Please review these first.

To walk through the process of adding a higher-order function, we will add a hypothetical `demo()` that takes in a real function of one variable, the derivative of that function, and a parameter. `demo()` calls the function on the input parameter. The process of implementing `demo()` closely follows the case of implementing a higher-order function with a known gradient.

We will extend the language to support this Stan program:
```
functions {
  real f(real t) {
    return 0.5*t;
  }
  
  real df(real t) {
    return 0.5;
  }
}
data { 
  int<lower=0> N; 
  int<lower=0,upper=1> y[N];
}
parameters {
  real<lower=0,upper=1.0> theta;
} 
model {
  real half_theta;
  half_theta = demo(f, df, theta);

  theta ~ beta(1,1);
  for (n in 1:N) 
    y[n] ~ bernoulli(half_theta);
}

```

# The Stan grammar

Editing the grammar involves editing these files:
- term_grammar.hpp
- term_grammar_def.hpp
- semantic_actions.hpp
- semantic_actions_def.cpp

The Stan language is parsed by Boost.Spirit. In Boost.Spirit, parsers are sets of [rules](https://theboostcpplibraries.com/boost.spirit-rules).

To add `demo()` to the Stan language, we need to create a rule to parse its presence in our programs. The rules of the Stan grammar are declared in `term_grammar.hpp` and defined in `term_grammar_def.hpp`.

We add a declaration to `term_grammar.hpp`:
```
boost::spirit::qi::rule<Iterator,
                        demo(scope),
                        whitespace_grammar<Iterator> >
demo_r;
```

We need to edit `term_grammar_def.hpp` in to three places. First,
```
BOOST_FUSION_ADAPT_STRUCT(stan::lang::demo,
                          (std::string, system_rhs_name_)
                          (std::string, system_jac_name_)
                          (stan::lang::expression, theta_) )
```
// TODO: better explain `BOOST_FUSION_ADAPT_STRUCT`

Next, define the rule used to parse the `demo` and its inputs:
```
demo_r.name("expression");
demo_r
  %= lit("demo")
  >> lit('(')
  >> identifier_r             // rhs
  >> lit(',')
  >> identifier_r            // jac
  >> lit(',')
  >> expression_g(_r1)    // theta
  > lit(')')
    [validate_demo_f(boost::phoenix::ref(error_msgs_))];
```

This configures the `demo_r` rule to find a term that starts with the literal string `demo(`, is followed by two identifiers (names of the functions it accepts), and an expression that defines the input parameter to `theta`. This rule also invokes `validate_demo_f()`, a semantic action which we will create in `semantic_actions.hpp`.

Finally, we need to extend the rule `factor_r` to include the `demo_r` rule. Edit the rule to add the “demo” line:
```
factor_r.name("expression");
factor_r =
    integrate_1d_r(_r1)[assign_lhs_f(_val, _1)]
  | integrate_ode_control_r(_r1)[assign_lhs_f(_val, _1)]
  | integrate_ode_r(_r1)[assign_lhs_f(_val, _1)]
  | demo_r(_r1)[assign_lhs_f(_val, _1)]
  | algebra_solver_control_r(_r1)[assign_lhs_f(_val, _1)]
  | algebra_solver_r(_r1)[assign_lhs_f(_val, _1)]
…
```
// TODO: better explain what “factor” is

Semantic actions are actions taken by the parser when a grammar rule is matched. When the Stan parser matches an expression it calls actions to validate the inputs. Above, we instructed the parser to call `validate_demo_f()` when a match is made. Declare this action in `semantic_actions.hpp`:

```
struct validate_demo : phoenix_functor_unary {
    void operator()(std::ostream &error_msgs) const;
};
extern boost::phoenix::function<validate_demo>
    validate_demo_f;
```

This validation function is intended as a stub. It only takes a stream to write messages, and not the inputs which would be needed in order to validate. See the other functions in this file for examples of functions that perform validation. 

The validation function, and other actions taken when `demo_r` is matched are defined in `semantic_actions_def.hpp` which we need to edit in three places. 

`demo()` returns a value, so we need to extend the template `assign_lhs()`:

```
template void assign_lhs::operator()(expression &, const demo &) const;
```

We implement the validation function:
```
void validate_demo::
operator()(std::ostream &error_msgs) const {
  error_msgs << "You're using the higher order function demo.\n" << std::endl;
}
boost::phoenix::function<validate_demo> validate_demo_f;
```

Lastly, we need to add a visitor at the end of the file:
```
bool data_only_expression::operator()(const demo &x) const {
  return boost::apply_visitor(*this, x.theta_.expr_);
}
```
// TODO: don’t know what this does.

# The abstract syntax tree (AST)

When we’re finished extending Stan to contain `demo()`, the function will be a node within the AST of the Stan language. Each node within the AST has data that is used in the code generation steps that follow parsing. To add `demo` we need to create or edit the following files:

- demo.hpp
- demo_def.hpp
- Ast.hpp
- Ast_def.cpp
- expression.hpp
- expression_def.hpp

First create the files `./stan/lang/ast/node/demo.hpp` and `./stan/lang/ast/node/demo_def.hpp`:

Demo.hpp:
```
namespace stan {
    namespace lang {

        struct expression;

        /**
         * Structure for demo()
         */
        struct demo {

            /**
             * Name of the RHS system.
             */
            std::string system_rhs_name_;

            /**
             * Name of the JAC system.
             */
            std::string system_jac_name_;

            /**
             * Input number.
             */
            expression theta_;


            /**
             * Construct a default demo.
             */
            demo();

            /**
             * Construt demo with the specified values.
             *
             * @param system_rhs_name name of function evaluated by demo()
             * @param system_jac_name name of derivative of system_rhs_name with respect to theta
             * @param theta parameter
             */
            idemo(const std::string& system_rhs_name,
                  const std::string& system_jac_name,
                  const expression& theta);

        };
    }
}
```

```
namespace stan {
    namespace lang {

        demo::demo() { }

        demo::demo(
                const std::string& system_rhs_name,
                const std::string& system_jac_name,
                const expression& theta)
                : system_rhs_name_(system_rhs_name),
                  system_jac_name_(system_jac_name),
                  theta_(theta) {
        }

    }
}
```

These files need to be imported in `ast.hpp` and `ast_def.hpp` respectively.

```
#include <stan/lang/ast/node/demo.hpp>
```

```
#include <stan/lang/ast/node/demo_def.hpp>
```

By adding, `demo`, we’ve changed the definition of `expression` and need to update it in `expression.hpp` and `expression_def.hpp`.

`expression.hpp` needs to be edited in three places:

1. Near the top of the file, add `struct demo;`
2. In the struct defining `expression`
    1. Add `boost::recursive_wrapper<demo>` to the `typedef` of `expression_t`
    2. Add `expression(const demo& expr);` to the list of specializations of `expression`

# The expression generator

For each node within the AST of a parsed Stan program, the compiler needs to generate code to evaluate the log probability of the model. The code we generate here references the symbol `demo` in `stan::math` that we define in the last section.
- expression_visgen.hpp

`expression_visgen.hpp` has function call operators that operate on expressions. We need to add one for `demo`:

```
void operator()(const demo& fx) const {
  o_ << "demo"
     << "("
     << fx.system_rhs_name_
     << "_functor__(), "
     << fx.system_jac_name_
     << "_functor__(), ";
  generate_expression(fx.theta_, user_facing_, o_);
  o_ << ")";
}
```
Higher order functions are implemented as functors in the generated code. `XXX_functor__` returns a wrapper to the function `XXX` that is callable from inside the `demo` function used here. We define `demo` in `stan::math` in the last section.

This operator generates code for our sample program that looks like:
```
stan::math::assign(half_theta, demo(f_functor__(), df_functor__(), theta));
```
Which directly corresponds to the line, `half_theta = demo(f, df, theta);`. You can see the generated code by inspecting the headers crated by calling `make` (if using cmdstan) on the program after completing the changes in this tutorial.

# The AST visitors
All of these files must be edited. // TODO better discussion of visitors

- expression_bare_type_vis.hpp
- expression_bare_type_vis_def.hpp
- has_non_param_var_vis.hpp
- has_non_param_var_vis_def.hpp
- has_var_vis.hpp
- has_var_vis_def.hpp
- is_nil_vis.hpp
- is_nil_vis_def.hpp
- var_occurs_vis.hpp
- var_occurs_vis_def.hpp
- write_expression_vis.hpp
- write_expression_vis_def.hpp

# The Stan math functions
As in adding new functions to Stan that are regular functions, we need to add the symbols to `stan::math`. See [this page](https://github.com/stan-dev/math/wiki/Adding-a-new-function-with-known-gradients) for a complete discussion. The only details here are the handling of the function inputs. The files that must be edited are:

- `stan/math/rev/functor/demo.hpp`
- `stan/math/rev/mat.hpp`

`demo.hpp`:

```
namespace stan {
    namespace math {

        template <typename RHS, typename JAC>
        double demo(const RHS &rhs, const JAC &jac, double theta) {
            std::ostream* pstream__ = 0;

            return rhs(theta, pstream__);
        }

        template <typename RHS, typename JAC>
        var demo(const RHS &rhs, const JAC &jac, const var& theta) {
            std::ostream* pstream__ = 0;

            double theta_dbl = theta.val();
            double rhs_theta = rhs(theta_dbl, pstream__);
            double drhs_dtheta = jac(theta_dbl, pstream__);
            return var(new precomp_v_vari(rhs_theta, theta.vi_, drhs_dtheta));
        }

    }
}
```

We define two function specializations that depend on the type of theta and the return value of `demo`. When calculating the value only we use `double`, when calculating the gradient we use `var`. The functor signature adds an output stream which we supply here. 

Add an import to mat.hpp: `#include <stan/math/rev/mat/functor/demo.hpp>`

