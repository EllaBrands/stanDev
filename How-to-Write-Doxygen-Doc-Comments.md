First off, here's a link to the 

* [Doxygen manual](http://www.doxygen.nl/index.html) (doxygen.nl doxygen site)


#### Function doc

Here's an example of what the doc should look like, so I can make some concrete recommendations on style (all of which are standard for Javadoc and presumably doxygen):

```
/**
 * Return a copy of the specified standard vector in descending
 * order.  The order of duplicated items is not defined.  Infinite
 * values come before or after all other elements depending on sign.
 *
 * @tparam T type of elements contained in vector
 * @param[in] xs vector to order
 * @return copy of vector in descending order
 * @throw std::domain_error if any of the values are NaN
 */
 template <typename T>
 inline std::vector<T> sort_desc(std::vector<T> xs) {
   check_not_nan("sort_desc", "container argument", xs);
   std::sort(xs.begin(), xs.end(), std::greater<T>());
   return xs;
 }
```

#### Broad Layout 

* The start-comment marker `/**`, continue comment markers `*` and the end-comment marker and `*/` are layed out in order to keep the left-side margin of the text aligned while including the necessary `/**` to kick off the doxygen comment.

* There is a space separating the parameters/return/throw statements, with template parameters first, arguments next in the order they appear in the function, followed by the return (if any) and exceptions (if any).

* Start new paragraphs with `<p>` as it will make sure the HTML does the right thing.

#### What to Document

* The first sentence, which will be used for a summary in the higher level docs, should be a concise but precise one-sentence statement of what the function returns and what its side effects are if any, based on what the arguments are.  

* Sentences after the first sentence can be used to define terms used in the first sentence, clarify finer behavior, provide example usage, mathematical background, or whatever else you deem necessary.

* Arguments should be documented by function and use `@param[in]`, `param[out]`, or `param[in,out]` depending on wehther the parameter must be specified on input, is defined on output, or is specified on input and modified on output.

#### Formatting details

* [How to Write Javadoc Comments](http://www.oracle.com/technetwork/articles/java/index-137868.html) (Oracle site)

* In particular, note that the arguments are not capitalized and do not need to be complete sentences, so do not end in periods.  

* It's OK to just call a one-argument function's generic argument the argument, but if there are finer terms like "multiplicand" or what not, those are preferable.

* The documentation of the template parameters, parameters, return and exception cases should not be duplicated in the body of the doc, but more detail can be given in the body of the class.

#### Class Documentation

Here's an example:

```
/**
 * The <code>compl</code> class represents an imaginary number
 * in terms of a contained real and imaginary components whose memory
 * is managed in this class.
 *
 * @tparam T scalar type for real and imaginary components
 */
template <typename T>
class compl {
private:
  const T rc_;
  const T ic_;
public:
  /**
   * Construct a complex number from the specified real
   * and imaginary components.
   *
   * @param[in] rc real component
   * @param[in] ic imaginary component
   */
  compl(double rc, double ic) : rc_(rc), ic_(ic) { }
  
  /**
   * Construct a complex number from the specified real
   * component, with zero imaginary component.
   *
   * @param[in] rc real component
   */
  compl(double rc) : rc_(rc), ic_(0) { }
  
  /**
   * Return the real component of this complex
   * number.
   *
   * @return real component
   */
  T rc() const { return rc_; }

  /**
   * Return the imaginary component of this complex
   * number.
   *
   * @return imaginary component
   */
  T ic() const { return ic_; }
};
```

* Classes should be documented themselves at the point where the class is declared (not where it's defined if this is separated).

* The class doc should talk about what instances of the class do in broad terms, but should not document arguments to constructors, provide a complete list of methods, etc.  It can talk about implementation details and why to use the class.

* The word "this" is used to refer to the instance of the class on which a method acts.

* Public and protected constructors should be documented.

* Public and protected member variables should be documented.

* No need to document the private components of a class with doxygen as they are not part of the API;  comments for developers can be included in whatever form seems helpful.

* It's OK to leave the copy constructor implicit if it's generated implicitly.

#### Using LaTeX

* Inline LaTeX via `\f$ e^x \f$`

* Display LaTeX via

```
\f[
  f(x) = e^x
\f]
```

* Equation arrays via

```
\f{eqnarray*}{
  f(x) & = & a (x + 1)
       & = ^ a x + a
\f}
```

#### Referring to Code

* For generic code-type formatting, use the code element, as in `<code>foo(a, b, c)</code>`

* Doxygen supports grave accent symbol syntax markup using \`code\` syntax.

* To link to other bits of the API (member variables, functions, classes, etc.), see [linking to other code](http://www.doxygen.nl/manual/autolink.html) (doxygen.nl manual)

#### Other Special Commands

Doxygen provides a rich set of commands for formatting and linking external references.  See:

* [Guide to Doxygen Commands](http://www.doxygen.nl/manual/commands.html) (doxygen.nl doxygen manual)
 