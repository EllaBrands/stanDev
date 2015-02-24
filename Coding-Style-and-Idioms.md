### Substance

#### Indexing Containers

For container indexes, use:

* `size_t` for `std::vector<S>`,

* `int` for `Eigen::Matrix<S, R, C>`, and

* `T::size_type` for template params `T` that might be instantiated as either a `std::vector<S>` or `Eigen::Matrix<S, R, C>`.

One could argue we should always use `Eigen::Matrix<S, R, C>::size_type` for Eigen constructs just in case we ever decide that we want to change their default indexing from `int` to something else.  But `int` should be OK for now because if we change the index type for some reason, the compiler will let us find instances.

### Style

#### No Tabs in Code

Use two spaces for indentation.  There are emacs modes to enforce this.

#### 80-character Lines

Whenever possible, keep lines to 80 characters or less.  It makes it much easier to review pull requests, view diffs, etc.

#### Templates

For both template parameter lists and template argument lists, use spaces between parameters and arguments. As above for template argument lists: 

`Eigen::Matrix<S, R, C>::size_type` 

and for template parameter lists: 

```
template<typename T1, typename T2>
struct foo {
...
};
```