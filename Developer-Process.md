**Rule 1.** Branch `master` is in a releasable state.

**Rule 2.** All contributions come through pull requests. 

1. Anyone can [create a pull request](https://help.github.com/articles/creating-a-pull-request).

2. A pull request should be unit-tested, `make test-unit`, at the very least. If your code touches anything that goes into distributions, a pull request should test the distributions, `make test-distributions`.

3. Anyone can review a pull request and comment on it. When reviewing a pull request we want to verify:

- Jenkins will run tests.

- Code review.

- 










- We need to explain how things are labeled and milestoned in the issue tracker.
- Explain how to review a pull request.
- How to push the patch.