### Substance

#### Indexing Containers

For container indexes, use:

* `size_t` for `std::vector<S>`,

* `int` for `Eigen::Matrix<S,R,C>`, and

* `T::size_type` for template params `T` that might be instantiated as either a `std::vector<S>` or `Eigen::Matrix<S,R,C>`.

One could argue we should always use Eigen::Matrix<S,R,C>::size_type` for Eigen constructs just in case we ever decide that we want to change
their default indexing from int to size_t.  But I don't see us
doing that, so I wouldn't worry about it.  


### Style

#### No Tabs in Code

Use two spaces for indentation.  There are emacs modes to enforce this.

#### 80-character Lines

Whenever possible, keep lines to 80 characters or less.  It makes it much easier to review pull requests, view diffs, etc.
